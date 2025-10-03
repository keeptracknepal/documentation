# Complete Hybrid Authentication Workflow

## Overview
The KeepTrack system uses a **Hybrid Authentication Architecture** where Authentication Service handles login/token management and Foundation Service handles all business logic with local JWT validation.

## Service Configuration

### Authentication Service (Port 8302)
- **Purpose**: Login, logout, JWT token generation
- **Environment**: `authentication/.env`
- **JWT_SECRET**: Same as Foundation Service

### Foundation Service (Port 8300)
- **Purpose**: API endpoints, business logic, JWT validation
- **Environment**: `foundation/.env`
- **JWT_SECRET**: Same as Authentication Service

## Complete Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              USER INTERACTION                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND APPLICATION                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │   Web App       │  │  Mobile App     │  │  Other Clients  │  │   Admin     │ │
│  │   (Port 3000)   │  │  (React Native) │  │   (External)    │  │   Panel     │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            AUTHENTICATION FLOW                                │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    STEP 1: USER LOGIN                                   │   │
│  │                                                                         │   │
│  │  Frontend → POST /auth/login → Authentication Service (8302)          │   │
│  │  ↓                                                                     │   │
│  │  Authentication Service → Database (User Validation)                   │   │
│  │  ↓                                                                     │   │
│  │  Database → Authentication Service (User Data)                        │   │
│  │  ↓                                                                     │   │
│  │  Authentication Service → JWT Token Generation (using JWT_SECRET)     │   │
│  │  ↓                                                                     │   │
│  │  Authentication Service → Frontend (JWT Token + User Info)            │   │
│  │  ↓                                                                     │   │
│  │  Frontend → Store JWT Token Securely                                  │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              API REQUEST FLOW                                 │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    STEP 2: API REQUESTS                               │   │
│  │                                                                         │   │
│  │  Frontend → API Request (with JWT Token) → Foundation Service (8300)  │   │
│  │  ↓                                                                     │   │
│  │  Foundation Service → JWT Middleware (Local Validation)               │   │
│  │  ↓                                                                     │   │
│  │  JWT Middleware → Validate Token (using same JWT_SECRET)              │   │
│  │  ↓                                                                     │   │
│  │  If Valid → Foundation Service → Business Logic                        │   │
│  │  ↓                                                                     │   │
│  │  Business Logic → Database (Business Operations)                      │   │
│  │  ↓                                                                     │   │
│  │  Database → Foundation Service (Business Data)                       │   │
│  │  ↓                                                                     │   │
│  │  Foundation Service → Frontend (API Response)                         │   │
│  │                                                                         │   │
│  │  If Invalid → Foundation Service → 401/403 Error                     │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Detailed Service Communication

### 1. Authentication Service (Port 8302)

#### **Endpoints:**
```
POST /auth/login          - User login (returns JWT)
POST /auth/logout         - User logout (invalidates JWT)
GET  /auth/me            - Current user info
GET  /auth/permissions   - User permissions
GET  /connection/test     - Database health check
GET  /connection/stats    - Connection statistics
POST /connection/rpc      - Database function calls
```

#### **Responsibilities:**
- ✅ User credential validation
- ✅ JWT token generation (signed with JWT_SECRET)
- ✅ User information management
- ✅ Session management
- ❌ No API request forwarding
- ❌ No business logic processing

### 2. Foundation Service (Port 8300)

#### **Endpoints:**
```
GET  /v1/dashboard/*      - Dashboard endpoints (JWT required)
GET  /v1/assets/*         - Asset management (JWT required)
GET  /v1/settings/*       - Settings management (JWT required)
GET  /v1/branches/*       - Branch management (JWT required)
GET  /connection/*        - Database connection endpoints
```

#### **Responsibilities:**
- ✅ JWT token validation (using same JWT_SECRET)
- ✅ Business logic processing
- ✅ API endpoint management
- ✅ Database operations
- ❌ No user login/logout
- ❌ No JWT token generation

## JWT Token Flow

### **Token Generation (Authentication Service):**
```go
// Authentication Service signs JWT with JWT_SECRET
token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
    "user_id": user.ID,
    "email": user.Email,
    "exp": time.Now().Add(time.Hour * 24).Unix(),
})
tokenString, err := token.SignedString([]byte(os.Getenv("JWT_SECRET")))
```

### **Token Validation (Foundation Service):**
```go
// Foundation Service validates JWT with same JWT_SECRET
token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
    return []byte(os.Getenv("JWT_SECRET")), nil
})
```

## Environment Configuration

### **Authentication Service (.env):**
```bash
JWT_SECRET=your-jwt-secret-here
AUTH_PORT=8302
DATABASE_URL=postgresql://...
```

### **Foundation Service (.env):**
```bash
JWT_SECRET=your-jwt-secret-here
PORT=8300
DATABASE_URL=postgresql://...
```

## Frontend Integration

### **Login Flow:**
```javascript
// Frontend login request
const response = await fetch('http://127.0.0.1:8302/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});

const data = await response.json();
if (data.success) {
  localStorage.setItem('jwt_token', data.token);
  // Store user info
}
```

### **API Request Flow:**
```javascript
// Frontend API request with JWT
const token = localStorage.getItem('jwt_token');
const response = await fetch('http://127.0.0.1:8300/v1/dashboard', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## Security Model

### **JWT Secret Sharing:**
- Both services use the **same JWT_SECRET**
- Authentication Service **signs** tokens
- Foundation Service **validates** tokens locally
- **No external API calls** for token validation

### **Request Flow:**
1. **Login**: Frontend → Authentication Service → JWT Token
2. **API**: Frontend → Foundation Service (with JWT) → Business Logic
3. **No Direct Communication**: Authentication Service ↔ Foundation Service

## Benefits of This Architecture

### **Performance:**
- ✅ Fast API calls (no gateway overhead)
- ✅ Local JWT validation (no external calls)
- ✅ Direct service communication

### **Security:**
- ✅ JWT tokens cryptographically signed
- ✅ Shared secret for validation
- ✅ No token exposure in logs

### **Scalability:**
- ✅ Each service can scale independently
- ✅ Clear separation of concerns
- ✅ Simple deployment

### **Maintainability:**
- ✅ Authentication logic centralized
- ✅ Business logic isolated
- ✅ Easy to debug and test

## Summary

**Yes, having the same JWT_SECRET in both services is enough!** This hybrid architecture provides:

1. **Authentication Service**: Handles login and JWT token generation
2. **Foundation Service**: Handles API requests with local JWT validation
3. **No Direct Communication**: Services don't talk to each other
4. **Shared JWT Secret**: Both services use the same secret for signing/validation
5. **Frontend Integration**: Simple two-endpoint communication

This setup is **production-ready**, **secure**, and **performant**!
