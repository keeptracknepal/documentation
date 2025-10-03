# JWT Authentication System - Setup Guide

## 🚀 **Service Setup Overview**

This guide covers the complete setup of the JWT authentication system with both Authentication and Foundation services.

## 🔐 **Authentication Service Setup**

### **Directory Structure:**
```
authentication/
├── connection/              # Database connections
│   └── supabase.go         # Supabase connection
├── routes/                  # Route configuration
│   └── routes.go           # Main route setup
├── main.go                 # Service entry point
├── go.mod                  # Go module dependencies
└── v1/                     # API version 1
    ├── login/              # User authentication
    │   ├── dto/            # Login DTOs
    │   ├── handler/        # Login handlers
    │   ├── models/         # Login models
    │   ├── repository/     # Login repositories
    │   ├── service/        # Login services
    │   └── login.module.go # Login module
    ├── permission/         # User permissions
    │   ├── dto/            # Permission DTOs
    │   ├── handler/        # Permission handlers
    │   ├── models/          # Permission models
    │   ├── repository/     # Permission repositories
    │   ├── service/        # Permission services
    │   └── permission.module.go # Permission module
    └── guard/jwt/          # JWT security operations
        ├── dto/            # JWT DTOs
        ├── handler/        # JWT handlers
        ├── models/         # JWT models
        ├── repository/     # JWT repositories
        ├── service/        # JWT services
        └── jwt.module.go   # JWT module
```

### **Environment Configuration:**
```bash
# authentication/.env
DATABASE_URL=postgresql://postgres.nhpxrwymvdwdovbexgvk:HeroBudathoki@aws-1-eu-west-2.pooler.supabase.com:6543/postgres?sslmode=require
JWT_SECRET=your-secure-jwt-secret-key-here
PORT=3001
```

> ⚠️ **SECURITY WARNING**: Never commit JWT secrets to version control! Use environment variables and secure secret management in production.

### **Route Configuration:**
```go
// authentication/routes/routes.go
func SetupAuthenticationRoutes(app *fiber.App, db *connection.AuthenticationDatabase) {
    v1 := app.Group("/v1")
    
    // Setup modules
    permissionImpl := service.NewPermissionService(db)
    loginModule := login.NewLoginModule(db.GetDB(), permissionImpl)
    permissionModule := permission.NewPermissionModule(db)
    jwtModule := jwt.NewJWTModule(db)
    
    // Register routes
    loginRoutes := loginModule.GetRoutes()
    for _, route := range loginRoutes {
        v1.Add(route.Method, route.Path, route.Handler)
    }
    
    permissionModule.RegisterRoutes(v1)
    jwtModule.RegisterRoutes(v1)
}
```

### **Module Dependencies:**
- **Login Module**: Depends on Permission Service for token generation
- **Permission Module**: Independent user permission management
- **JWT Module**: Independent JWT security operations

## 🛡️ **Foundation Service Setup**

### **Directory Structure:**
```
foundation/
├── connection/              # Database connections
│   └── database.go         # Database connection
├── middleware/              # JWT middleware
│   └── jwt.go              # JWT validation middleware
├── routes/                  # Route configuration
│   └── routes.go           # Main route setup
├── main.go                 # Service entry point
├── go.mod                  # Go module dependencies
├── .env                    # Environment variables
└── v1/                     # API version 1
    ├── dashboard/          # Dashboard functionality
    │   ├── assets/         # Asset management
    │   ├── branches/       # Branch operations
    │   ├── positions/      # Position management
    │   └── settings/       # User settings
    └── guard/              # Security operations
        ├── guard.module.go # Guard module wrapper
        └── jwt/            # JWT validation
            ├── dto/        # JWT DTOs
            ├── handler/    # JWT handlers
            ├── models/     # JWT models
            ├── repository/ # JWT repositories
            ├── service/    # JWT services
            └── jwt.module.go # JWT module
```

### **Environment Configuration:**
```bash
# foundation/.env
JWT_SECRET=your-secure-jwt-secret-key-here
```

> ⚠️ **SECURITY WARNING**: Never commit JWT secrets to version control! Use environment variables and secure secret management in production.

### **Route Configuration:**
```go
// foundation/routes/routes.go
func SetupRoutes(app *fiber.App, db *connection.Database) {
    // Setup modules
    dashboardModule := dashboard.NewDashboardModule(db.GetDB(), nil)
    guardModule := guard.NewGuardModule(db)
    
    // Create v1 API group with JWT middleware
    v1 := app.Group("/v1")
    v1.Use(middleware.JWTMiddleware()) // Apply JWT middleware to all v1 routes
    
    // Register routes
    dashboardModule.RegisterRoutes(v1)
    guardModule.RegisterRoutes(v1)
}
```

### **JWT Middleware Setup:**
```go
// foundation/middleware/jwt.go
func JWTMiddleware() fiber.Handler {
    return func(c *fiber.Ctx) error {
        // JWT validation logic
        // Request counting
        // User blocking
    }
}
```

## 🗄️ **Database Setup**

### **Authentication Service Database:**
```sql
-- authentication schema
CREATE SCHEMA IF NOT EXISTS authentication;

-- JWT request counters table
CREATE TABLE IF NOT EXISTS authentication.jwt_request_counters (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(255) NOT NULL UNIQUE,
    failed_count INTEGER DEFAULT 0,
    last_attempt TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    is_blocked BOOLEAN DEFAULT FALSE,
    blocked_until TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### **Foundation Service Database:**
```sql
-- core schema
CREATE SCHEMA IF NOT EXISTS core;

-- Dashboard tables
CREATE TABLE IF NOT EXISTS core.assets (...);
CREATE TABLE IF NOT EXISTS core.branches (...);
CREATE TABLE IF NOT EXISTS core.positions (...);
```

## 🔧 **Module Configuration**

### **Authentication Service Modules:**

#### **1. Login Module:**
```go
// authentication/v1/login/login.module.go
type LoginModule struct {
    handler *handler.LoginHandler
}

func NewLoginModule(db *sql.DB, permission models.PermissionInterface) *LoginModule {
    // Module initialization
}
```

#### **2. Permission Module:**
```go
// authentication/v1/permission/permission.module.go
type PermissionModule struct {
    service *service.PermissionService
    handler *handler.PermissionHandler
}

func NewPermissionModule(db *connection.AuthenticationDatabase) *PermissionModule {
    // Module initialization
}
```

#### **3. JWT Module:**
```go
// authentication/v1/guard/jwt/jwt.module.go
type JWTModule struct {
    service *service.JWTService
    handler *handler.JWTHandler
}

func NewJWTModule(db *connection.AuthenticationDatabase) *JWTModule {
    // Module initialization
}
```

### **Foundation Service Modules:**

#### **1. Guard Module:**
```go
// foundation/v1/guard/guard.module.go
type GuardModule struct {
    jwtModule *jwt.JWTModule
}

func NewGuardModule(db *connection.Database) *GuardModule {
    // Module initialization
}
```

#### **2. JWT Module:**
```go
// foundation/v1/guard/jwt/jwt.module.go
type JWTModule struct {
    service *service.JWTService
    handler *handler.JWTHandler
}

func NewJWTModule(db *connection.Database) *JWTModule {
    // Module initialization
}
```

## 🚀 **Service Startup**

### **Authentication Service:**
```bash
cd authentication
go mod tidy
go run main.go
```

### **Foundation Service:**
```bash
cd foundation
go mod tidy
go run main.go
```

## 🔐 **Security Configuration**

### **JWT Secret:**
- **Shared Secret**: Both services use the same `JWT_SECRET`
- **Environment Variable**: Stored in `.env` files
- **Security**: Never commit secrets to version control

### **Request Counter:**
- **Threshold**: 3 failed attempts
- **Blocking**: User blocked after threshold
- **Recovery**: Automatic unblocking after timeout

### **Token Management:**
- **Access Tokens**: Short-lived (15 minutes)
- **Refresh Tokens**: Long-lived (7 days)
- **Validation**: Real-time verification

## 📊 **Monitoring Setup**

### **JWT Statistics:**
- Token validation counts
- Failed validation attempts
- Blocked user statistics
- Performance metrics

### **Security Monitoring:**
- Request counter tracking
- User blocking events
- Token revocation events
- Authentication failures

## 🧪 **Testing Setup**

### **Unit Tests:**
```bash
# Authentication Service
cd authentication
go test ./...

# Foundation Service
cd foundation
go test ./...
```

### **Integration Tests:**
```bash
# Test JWT validation
curl -X POST http://localhost:3001/v1/guard/jwt/validate \
  -H "Content-Type: application/json" \
  -d '{"token": "your-jwt-token"}'
```

## 🔄 **Deployment Setup**

### **Production Environment:**
- **Authentication Service**: `https://authentication.keeptrack.velonovo.com`
- **Foundation Service**: `https://foundation.keeptrack.velonovo.com`
- **Database**: Supabase PostgreSQL
- **Load Balancer**: Cloudflare/CDN

### **Environment Variables:**
```bash
# Production
JWT_SECRET=production-secret-key
DATABASE_URL=production-database-url
PORT=3001
```