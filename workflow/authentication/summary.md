# Authentication Service Summary

## Overview
The KeepTrack Authentication Service is a dedicated microservice responsible for user authentication, authorization, and session management across the entire KeepTrack ecosystem using a **Hybrid Authentication Architecture**.

## Service Architecture

### Core Components
- **Authentication Service**: Standalone service handling login and JWT token generation
- **Foundation Service**: Handles API requests with local JWT validation
- **Database Connection**: Dedicated Supabase connection for authentication operations
- **JWT Management**: Token generation, validation, and refresh
- **Guard System**: Permission-based access control

### Service Structure
```
authentication/
├── connection/          # Database connection (Supabase)
├── server/             # HTTP server setup
├── routes/             # API route definitions
├── v1/                 # Authentication modules
│   ├── guard/          # Permission management
│   ├── login/          # User authentication
│   └── authentication.module.go
├── main.go             # Service entry point
└── go.mod              # Dependencies
```

## Production Endpoints

### Authentication Service
```
https://authentication.keeptrack.velonovo.com/v1/login          - User authentication
https://authentication.keeptrack.velonovo.com/v1/logout         - User logout
https://authentication.keeptrack.velonovo.com/v1/me            - Current user info
https://authentication.keeptrack.velonovo.com/v1/permissions   - User permissions
https://authentication.keeptrack.velonovo.com/v1/guard/me      - Guard permissions
```

### Foundation Service
```
https://foundation.keeptrack.velonovo.com/v1/dashboard/*      - Dashboard endpoints
https://foundation.keeptrack.velonovo.com/v1/assets/*         - Asset management
https://foundation.keeptrack.velonovo.com/v1/settings/*       - Settings management
https://foundation.keeptrack.velonovo.com/v1/branches/*       - Branch management
```

## Hybrid Authentication Flow

The authentication system follows a hybrid approach where:
1. **Authentication Service** handles login and JWT token generation
2. **Foundation Service** handles API requests with local JWT validation
3. **No Direct Communication** between services
4. **Shared JWT Secret** for token signing and validation

## Service Communication

The system uses a simple two-step process:
1. **Login Flow**: Frontend → Authentication Service → JWT Token
2. **API Flow**: Frontend → Foundation Service (with JWT) → Business Logic

## Key Features

### 1. User Authentication
- **Login/Logout**: Secure user authentication
- **Password Management**: Bcrypt password hashing
- **Session Management**: JWT-based sessions
- **Token Refresh**: Automatic token renewal

### 2. Authorization & Permissions
- **Role-Based Access**: User role management
- **Permission System**: Granular permission control
- **Guard Middleware**: Request-level authorization
- **Access Lists**: JSON-based permission structure

### 3. Security Features
- **JWT Tokens**: Secure token-based authentication
- **Password Hashing**: Bcrypt encryption
- **CORS Support**: Cross-origin request handling
- **Rate Limiting**: Request throttling capabilities

## Environment Configuration

### Authentication Service (.env)
```bash
JWT_SECRET=your-jwt-secret-here
AUTH_PORT=8302
DATABASE_URL=postgresql://...
```

### Foundation Service (.env)
```bash
JWT_SECRET=your-jwt-secret-here
PORT=8300
DATABASE_URL=postgresql://...
```

## Frontend Integration

### Login Flow
```javascript
// Frontend login request
const response = await fetch('https://authentication.keeptrack.velonovo.com/v1/login', {
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

### API Request Flow
```javascript
// Frontend API request with JWT
const token = localStorage.getItem('jwt_token');
const response = await fetch('https://foundation.keeptrack.velonovo.com/v1/dashboard', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## Security Model

### JWT Secret Sharing
- Both services use the **same JWT_SECRET**
- Authentication Service **signs** tokens
- Foundation Service **validates** tokens locally
- **No external API calls** for token validation

### Request Flow
1. **Login**: Frontend → Authentication Service → JWT Token
2. **API**: Frontend → Foundation Service (with JWT) → Business Logic
3. **No Direct Communication**: Authentication Service ↔ Foundation Service

## Benefits

### Performance
- ✅ Fast API calls (no gateway overhead)
- ✅ Local JWT validation (no external calls)
- ✅ Direct service communication

### Security
- ✅ JWT tokens cryptographically signed
- ✅ Shared secret for validation
- ✅ No token exposure in logs

### Scalability
- ✅ Each service can scale independently
- ✅ Clear separation of concerns
- ✅ Simple deployment

### Maintainability
- ✅ Authentication logic centralized
- ✅ Business logic isolated
- ✅ Easy to debug and test

## Service Dependencies
- **Go Fiber**: HTTP framework
- **JWT**: Token management
- **Bcrypt**: Password hashing
- **PostgreSQL**: Database driver
- **CORS**: Cross-origin support

## Deployment
- **Docker Support**: Containerized deployment
- **Port Configuration**: Authentication (8302), Foundation (8300)
- **Health Checks**: Service monitoring
- **Logging**: Comprehensive request logging

## Summary

This hybrid authentication architecture provides:
- **Authentication Service**: Handles login and JWT token generation
- **Foundation Service**: Handles API requests with local JWT validation
- **No Direct Communication**: Services don't talk to each other
- **Shared JWT Secret**: Both services use the same secret for signing/validation
- **Frontend Integration**: Simple two-endpoint communication

The system is **production-ready**, **secure**, and **performant** with clear separation of concerns and scalable architecture.
