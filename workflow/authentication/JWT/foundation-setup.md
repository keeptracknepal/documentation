# Foundation Service - Guard Integration Setup

## 🛡️ **Guard Integration Overview**

The Foundation Service uses a centralized JWT guard system that protects all API routes. The guard integration is configured at the main module level and automatically applies to all sub-routes.

## 🏗️ **Guard Architecture**

### **Centralized Guard System:**
```
foundation/
├── middleware/              # JWT middleware (removed - using guard module)
├── routes/                  # Main route configuration
└── v1/
    ├── guard/              # Guard module with JWT functionality
    │   ├── guard.module.go # Main guard module wrapper
    │   └── jwt/            # JWT validation and security
    └── dashboard/          # Dashboard modules (protected by guard)
        ├── assets/         # Asset management
        ├── branches/       # Branch operations
        ├── positions/      # Position management
        └── settings/       # User settings
```

## 🔧 **Guard Integration Setup**

### **1. Main Route Configuration**

```go
// foundation/routes/routes.go
package routes

import (
	"github.com/gofiber/fiber/v2"
	"github.com/keeptracknepal/foundation/connection"
	"github.com/keeptracknepal/foundation/v1/dashboard"
	"github.com/keeptracknepal/foundation/v1/guard"
)

// SetupRoutes configures all foundation routes with guard protection
func SetupRoutes(app *fiber.App, db *connection.Database) {
	// Setup database connection routes
	connection.DatabaseConnectionRoutes(app, db)

	// Setup dashboard module
	dashboardModule := dashboard.NewDashboardModule(db.GetDB())

	// Setup guard module
	guardModule := guard.NewGuardModule(db)

	// Create v1 API group with JWT middleware
	v1 := app.Group("/v1")
	v1.Use(guardModule.GetJWTModule().GetJWTMiddleware()) // Apply JWT middleware to all v1 routes

	// Register dashboard routes (now protected by JWT middleware)
	dashboardModule.RegisterRoutes(v1)

	// Register guard routes (includes JWT under /v1/guard/jwt)
	guardModule.RegisterRoutes(v1)
}
```

### **2. Guard Module Configuration**

```go
// foundation/v1/guard/guard.module.go
package guard

import (
	"github.com/gofiber/fiber/v2"
	"github.com/keeptracknepal/foundation/connection"
	"github.com/keeptracknepal/foundation/v1/guard/jwt"
)

// GuardModule wires together the guard service and handler layers
type GuardModule struct {
	jwtModule *jwt.JWTModule
}

// NewGuardModule creates a new guard module
func NewGuardModule(db *connection.Database) *GuardModule {
	jwtModule := jwt.NewJWTModule(db)

	return &GuardModule{
		jwtModule: jwtModule,
	}
}

// RegisterRoutes registers guard routes with the router
func (gm *GuardModule) RegisterRoutes(router fiber.Router) {
	guardRoutes := router.Group("/guard")
	
	// Register JWT routes under /guard/jwt
	gm.jwtModule.RegisterRoutes(guardRoutes)
}

// GetJWTModule returns the JWT module
func (gm *GuardModule) GetJWTModule() *jwt.JWTModule {
	return gm.jwtModule
}
```

### **3. JWT Module Configuration**

```go
// foundation/v1/guard/jwt/jwt.module.go
package jwt

import (
	"github.com/gofiber/fiber/v2"
	"github.com/keeptracknepal/foundation/connection"
	"github.com/keeptracknepal/foundation/v1/guard/jwt/handler"
	"github.com/keeptracknepal/foundation/v1/guard/jwt/service"
)

// JWTModule wires together the JWT service and handler layers
type JWTModule struct {
	service *service.JWTService
	handler *handler.JWTHandler
}

// NewJWTModule creates a new JWT module
func NewJWTModule(db *connection.Database) *JWTModule {
	service := service.NewJWTService(db)
	handler := handler.NewJWTHandler(service)

	return &JWTModule{
		service: service,
		handler: handler,
	}
}

// RegisterRoutes registers JWT routes with the router
func (jm *JWTModule) RegisterRoutes(router fiber.Router) {
	jwtRoutes := router.Group("/jwt")

	// JWT validation routes
	jwtRoutes.Post("/validate", jm.handler.ValidateToken)
	jwtRoutes.Post("/refresh", jm.handler.RefreshToken)
	jwtRoutes.Post("/revoke", jm.handler.RevokeToken)
	jwtRoutes.Get("/stats", jm.handler.GetJWTStats)
	jwtRoutes.Get("/blocked-users", jm.handler.GetBlockedUsers)
}

// GetJWTMiddleware returns JWT middleware with request counting
func (jm *JWTModule) GetJWTMiddleware() fiber.Handler {
	return jm.service.JWTMiddlewareWithCounter()
}
```

## 🏢 **Dashboard Module Guard Integration**

### **Main Dashboard Module Setup**

```go
// foundation/v1/dashboard/dashboard.module.go
package dashboard

import (
	"database/sql"
	"github.com/gofiber/fiber/v2"
	"github.com/keeptracknepal/foundation/v1/dashboard/assets"
	"github.com/keeptracknepal/foundation/v1/dashboard/branches"
	"github.com/keeptracknepal/foundation/v1/dashboard/positions"
	"github.com/keeptracknepal/foundation/v1/dashboard/settings"
)

// Route represents a route configuration
type Route struct {
	Method  string
	Path    string
	Handler fiber.Handler
}

// DashboardModule represents the dashboard module configuration
type DashboardModule struct {
	AssetsModule    *assets.AssetsModule
	BranchesModule  *branches.BranchesModule
	PositionsModule *positions.PositionsModule
	SettingsModule  *settings.SettingsModule
}

// NewDashboardModule creates a new dashboard module
func NewDashboardModule(db *sql.DB) *DashboardModule {
	assetsModule := assets.NewAssetsModule(db)
	branchesModule := branches.NewBranchesModule(db)
	positionsModule := positions.NewPositionsModule(db)
	settingsModule := settings.NewSettingsModule(db)

	return &DashboardModule{
		AssetsModule:    assetsModule,
		BranchesModule:  branchesModule,
		PositionsModule: positionsModule,
		SettingsModule:  settingsModule,
	}
}

// RegisterRoutes registers all dashboard routes with the router
func (dm *DashboardModule) RegisterRoutes(router fiber.Router) {
	// Register assets routes
	dm.AssetsModule.RegisterRoutes(router)
	
	// Register branches routes
	dm.BranchesModule.RegisterRoutes(router)
	
	// Register positions routes
	dm.PositionsModule.RegisterRoutes(router)
	
	// Register settings routes
	dm.SettingsModule.RegisterRoutes(router)
}
```

### **Sub-Module Guard Integration**

Each sub-module (assets, branches, positions, settings) is automatically protected by the JWT middleware applied at the main route level.

#### **Assets Module Example:**

```go
// foundation/v1/dashboard/assets/assets.module.go
package assets

import (
	"database/sql"
	"github.com/gofiber/fiber/v2"
	"github.com/keeptracknepal/foundation/v1/dashboard/assets/create"
	"github.com/keeptracknepal/foundation/v1/dashboard/assets/list"
)

// Route represents a route configuration
type Route struct {
	Method  string
	Path    string
	Handler fiber.Handler
}

// AssetsModule represents the assets module configuration
type AssetsModule struct {
	ListModule   *list.ListModule
	CreateModule *create.CreateModule
}

// NewAssetsModule creates a new assets module
func NewAssetsModule(db *sql.DB) *AssetsModule {
	listModule := list.NewListModule(db)
	createModule := create.NewCreateModule(db)

	return &AssetsModule{
		ListModule:   listModule,
		CreateModule: createModule,
	}
}

// RegisterRoutes registers all assets routes with the router
func (am *AssetsModule) RegisterRoutes(router fiber.Router) {
	// Register list routes
	am.ListModule.RegisterRoutes(router)

	// Register create routes
	am.CreateModule.RegisterRoutes(router)
}
```

## 🔐 **JWT Middleware Protection**

### **Automatic Protection:**
- **All `/v1` routes** are automatically protected by JWT middleware
- **No manual authentication** needed in individual modules
- **Centralized security** through the guard module
- **Request counting** and user blocking handled automatically

### **JWT Middleware Features:**
- **Token Validation**: Validates JWT tokens on every request
- **Request Counting**: Tracks failed validation attempts
- **User Blocking**: Blocks users after 3 failed attempts
- **Context Injection**: Adds user information to request context

### **Accessing User Information:**

```go
// In any handler, access user information from context
func (h *SomeHandler) SomeEndpoint(c *fiber.Ctx) error {
	// Get user information from JWT middleware
	userID := c.Locals("user_id").(string)
	email := c.Locals("email").(string)
	permissions := c.Locals("permissions").([]string)
	
	// Use user information in business logic
	// ...
}
```

## 🚀 **Production Endpoints**

### **Protected Dashboard Endpoints:**
- **Assets**: `https://foundation.keeptrack.velonovo.com/v1/dashboard/assets/*`
- **Branches**: `https://foundation.keeptrack.velonovo.com/v1/dashboard/branches/*`
- **Positions**: `https://foundation.keeptrack.velonovo.com/v1/dashboard/positions/*`
- **Settings**: `https://foundation.keeptrack.velonovo.com/v1/dashboard/settings/*`

### **Guard Endpoints:**
- **JWT Validation**: `https://foundation.keeptrack.velonovo.com/v1/guard/jwt/validate`
- **JWT Stats**: `https://foundation.keeptrack.velonovo.com/v1/guard/jwt/stats`
- **Blocked Users**: `https://foundation.keeptrack.velonovo.com/v1/guard/jwt/blocked-users`

## 🔧 **Environment Configuration**

### **Foundation Service Environment:**
```bash
# foundation/.env
JWT_SECRET=your-secure-jwt-secret-key-here
DATABASE_URL=postgresql://postgres.nhpxrwymvdwdovbexgvk:HeroBudathoki@aws-1-eu-west-2.pooler.supabase.com:6543/postgres?sslmode=require
```

> ⚠️ **SECURITY WARNING**: Never commit JWT secrets to version control! Use environment variables and secure secret management in production.

## 📊 **Guard Integration Benefits**

### **Centralized Security:**
- **Single Point of Control**: All security logic in guard module
- **Consistent Protection**: All routes protected uniformly
- **Easy Maintenance**: Security updates in one place

### **Performance Optimization:**
- **Local JWT Validation**: No external service calls for validation
- **Request Counting**: Efficient tracking of failed attempts
- **Context Injection**: Fast access to user information

### **Scalability:**
- **Stateless Design**: No session storage required
- **Horizontal Scaling**: Multiple service instances supported
- **Load Balancing**: JWT tokens work across all instances

## 🧪 **Testing Guard Integration**

### **Test JWT Validation:**
```bash
# Test JWT validation endpoint
curl -X POST https://foundation.keeptrack.velonovo.com/v1/guard/jwt/validate \
  -H "Content-Type: application/json" \
  -d '{"token": "your-jwt-token"}'
```

### **Test Protected Endpoints:**
```bash
# Test protected dashboard endpoint
curl -X GET https://foundation.keeptrack.velonovo.com/v1/dashboard/assets/list \
  -H "Authorization: Bearer your-jwt-token"
```

### **Test JWT Stats:**
```bash
# Test JWT statistics
curl -X GET https://foundation.keeptrack.velonovo.com/v1/guard/jwt/stats
```

## 🔄 **Guard Integration Workflow**

### **Request Flow:**
1. **Client Request** → Foundation Service
2. **JWT Middleware** → Validates token and extracts user info
3. **Dashboard Module** → Processes business logic
4. **Response** → Returns data to client

### **Security Flow:**
1. **Token Validation** → Check JWT signature and expiration
2. **Request Counting** → Track failed attempts
3. **User Blocking** → Block users after threshold
4. **Context Injection** → Add user info to request context

This guard integration setup ensures that all dashboard modules and their sub-routes are automatically protected by JWT authentication without requiring individual module configuration.
