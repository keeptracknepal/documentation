# Authentication Architecture

## System Architecture Overview

### Hybrid Authentication Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web App       │    │  Mobile App     │    │  Other Services │
│   (Frontend)    │    │  (Frontend)     │    │  (External)    │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          │ LOGIN: Auth Service  │ LOGIN: Auth Service  │ LOGIN: Auth Service
          │ API: Foundation      │ API: Foundation      │ API: Foundation
          ▼                      ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                Authentication Service                          │
│                (Port 8400) - LOGIN ONLY                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐│
│  │   Login         │  │   Guard         │  │   JWT           ││
│  │   Module        │  │   Module        │  │   Management    ││
│  └─────────────────┘  └─────────────────┘  └─────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              LOGIN & TOKEN MANAGEMENT ONLY                 ││
│  │  • User Login/Logout                                       ││
│  │  • JWT Token Generation                                    ││
│  │  • User Information                                        ││
│  │  • Permission Management                                   ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
          │
          │ DIRECT API CALLS (with JWT)
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Foundation Service                          │
│                    (Port 8300) - API & JWT VALIDATOR          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐│
│  │   Dashboard     │  │   Assets        │  │   Settings       ││
│  │   Module        │  │   Module        │  │   Module         ││
│  └─────────────────┘  └─────────────────┘  └─────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              JWT VALIDATION & BUSINESS LOGIC               ││
│  │  • Local JWT Token Validation                              ││
│  │  • Permission Checking                                     ││
│  │  • Business Logic Processing                               ││
│  │  • Direct Response Generation                              ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
          │
          │ Database Connection
          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Supabase Database                           │
│                    (PostgreSQL)                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐│
│  │   Users         │  │   Permissions  │  │   Sessions      ││
│  │   Table         │  │   Table        │  │   Table         ││
│  └─────────────────┘  └─────────────────┘  └─────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## Service Components

### 1. Authentication Service (Port 8400) - LOGIN ONLY

#### Core Modules
- **Login Module**: User authentication and session management
- **Guard Module**: Permission-based access control
- **JWT Module**: Token generation and management

#### Primary Functionality
- **User Login**: Validates credentials and issues JWT tokens
- **User Logout**: Invalidates JWT tokens
- **User Information**: Provides user details and permissions
- **Token Management**: Generates JWT tokens (no validation for API calls)

#### API Endpoints
```
Authentication Service - LOGIN SERVICE ONLY
├── /v1/login           - User authentication (returns JWT)
├── /v1/logout          - User logout (invalidates JWT)
├── /v1/me             - Current user info
├── /v1/permissions    - User permissions
└── /v1/guard/me       - Guard permissions
```

### 2. Foundation Service (Port 8300) - API & JWT VALIDATOR

#### Core Modules
- **Dashboard Module**: Main dashboard functionality
- **Assets Module**: Asset management
- **Settings Module**: System settings
- **Branches Module**: Branch management
- **JWT Middleware**: Local JWT token validation

#### Security Model
- **Local JWT Validation**: Validates JWT tokens locally (no external calls)
- **Permission Checking**: Checks user permissions for each request
- **Business Logic**: Processes business requests after validation
- **Direct Access**: Frontend communicates directly with Foundation

#### API Endpoints
```
Foundation Service (Port 8300) - API & JWT VALIDATOR
├── /v1/dashboard/*      - Dashboard endpoints (JWT required)
├── /v1/assets/*         - Asset management (JWT required)
├── /v1/settings/*       - Settings management (JWT required)
├── /v1/branches/*       - Branch management (JWT required)
└── /connection/*        - Database connection endpoints
```

## Data Flow Architecture

### 1. User Login Flow
```
1. User Login Request
   ↓
2. Web/Mobile → Authentication Service
   ↓
3. Authentication Service → Database (User Validation)
   ↓
4. Database → Authentication Service (User Data)
   ↓
5. Authentication Service → JWT Token Generation
   ↓
6. Authentication Service → Web/Mobile (JWT Token)
   ↓
7. Web/Mobile → Store Token Securely
```

### 2. API Request Flow (Direct Communication)
```
1. User API Request
   ↓
2. Web/Mobile → Foundation Service (with JWT Token)
   ↓
3. Foundation Service → JWT Middleware (Local validation)
   ↓
4. If Valid → Foundation Service → Business Logic
   ↓
5. If Invalid → Foundation Service → 401/403 Response
   ↓
6. Foundation Service → Web/Mobile (Response)
```

### 3. Token Validation Flow (Local)
```
1. Foundation Service receives request with JWT
   ↓
2. Foundation Service validates JWT token locally (no external calls)
   ↓
3. If valid → Process business logic
   ↓
4. If invalid → Return 401/403 error
   ↓
5. Foundation Service returns response to frontend
```

## Security Architecture

### 1. Token-Based Authentication
- **JWT Tokens**: Secure, stateless authentication
- **Token Expiration**: Short-lived access tokens (15-30 minutes)
- **Refresh Tokens**: Long-lived refresh tokens for seamless experience
- **Token Rotation**: Automatic token refresh mechanism

### 2. Permission System
- **Role-Based Access**: User roles (admin, user, guest)
- **Permission Matrix**: Granular permission control
- **Access Lists**: JSON-based permission structure
- **Guard Middleware**: Request-level authorization

### 3. Database Security
- **Encrypted Connections**: SSL/TLS for database communication
- **Connection Pooling**: Optimized connection management
- **User Isolation**: Separate database users for each service
- **Audit Logging**: Comprehensive security event logging

## Service Communication

### 1. Internal Communication
- **Service Discovery**: Docker network communication
- **Load Balancing**: Multiple service instances
- **Health Checks**: Service availability monitoring
- **Circuit Breakers**: Fault tolerance mechanisms

### 2. External Communication
- **HTTPS**: Encrypted external communication
- **CORS**: Cross-origin resource sharing
- **Rate Limiting**: Request throttling
- **API Versioning**: Backward compatibility

## Deployment Architecture

### 1. Docker Containerization
```yaml
# Authentication Service
authentication:
  build: ./authentication
  ports: ["8400:8400"]
  environment:
    - DATABASE_URL=postgresql://...
    - JWT_SECRET=your-secret-key

# Foundation Service
foundation:
  build: ./foundation
  ports: ["8300:8300"]
  environment:
    - AUTH_SERVICE_URL=http://authentication:8400
    - DATABASE_URL=postgresql://...
```

### 2. Service Dependencies
- **Authentication Service**: Independent, no dependencies
- **Foundation Service**: Depends on Authentication Service
- **Web Application**: Depends on Foundation Service
- **Mobile Application**: Depends on Foundation Service

### 3. Scaling Strategy
- **Horizontal Scaling**: Multiple service instances
- **Load Balancing**: Request distribution
- **Database Scaling**: Connection pool optimization
- **Caching**: Redis for session management

## Monitoring and Observability

### 1. Health Monitoring
- **Service Health**: HTTP health check endpoints
- **Database Health**: Connection status monitoring
- **Token Health**: JWT validation monitoring
- **Performance Metrics**: Response time and throughput

### 2. Security Monitoring
- **Authentication Events**: Login/logout tracking
- **Permission Checks**: Access control monitoring
- **Security Alerts**: Suspicious activity detection
- **Audit Logs**: Comprehensive security logging

### 3. Business Metrics
- **User Activity**: Login patterns and usage
- **API Usage**: Endpoint usage statistics
- **Error Rates**: Service reliability metrics
- **Performance**: Response time analysis

This architecture provides a robust, scalable, and secure foundation for authentication across all KeepTrack services.
