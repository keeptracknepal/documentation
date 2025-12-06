# Go Backend - Server and Routes Standards

## Overview
This document defines the standards for server setup and route configuration in Go backend projects using GoFiber framework. The architecture emphasizes minimal server configuration with module-based route management.

## Core Principles

### 1. Minimal Server Configuration
- **Routes File**: Acts as a simple index of route modules only
- **Module Independence**: Each module handles its own initialization and dependencies
- **Clean Separation**: Server file only lists modules, no implementation details
- **Self-contained Modules**: Modules manage their own dependencies and route setup

### 2. Route Organization
- **Module-based**: Routes are organized by feature/module
- **Self-contained**: Each module knows its own routes
- **Versioned**: Routes are grouped by API version (e.g., `/v1`, `/v2`)
- **Documented**: Route details are documented in module files

## Server Structure

### Main Server File (`server/routes.go`)

The main server file should be a simple index of route modules. It should:
- Only list module setup calls
- One line per module
- No initialization logic
- No dependency management
- No route path definitions

```go
// ✅ Correct: Simple index of route modules
package server

import (
	"github.com/gofiber/fiber/v2"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/login"
	"github.com/youthcongress/authentication/v1/otp"
	passwordreset "github.com/youthcongress/authentication/v1/password-reset"
	"github.com/youthcongress/authentication/v1/register"
)

// SetupRoutes configures all application routes
// This file acts as an index of all route modules
func SetupRoutes(app *fiber.App, db *connection.Database) {
	v1 := app.Group("/v1")

	// Route modules index
	register.SetupRoutes(v1, db)
	login.SetupRoutes(v1, db)
	otp.SetupRoutes(v1, db)
	passwordreset.SetupRoutes(v1, db)
}
```

```go
// ❌ Incorrect: Too much detail in routes file
func SetupRoutes(app *fiber.App, db *connection.Database) {
	v1 := app.Group("/v1")

	// Initialize modules
	otpModule := otp.NewOTPModule(db.GetDB())
	registerModule := register.NewRegisterModule(db.GetDB(), otpModule.Service)
	loginModule := login.NewLoginModule(db.GetDB())
	passwordResetModule := passwordreset.NewPasswordResetModule(db.GetDB(), otpModule.Service)

	// Configure module dependencies
	otpModule.SetCredentialRepository(registerModule.Repo)

	// Register module routes
	registerModule.SetupRoutes(v1)
	loginModule.SetupRoutes(v1)
	otpModule.SetupRoutes(v1)
	passwordResetModule.SetupRoutes(v1)
}
```

## Module Route Setup

### Module Setup Function Pattern

Each module should export a `SetupRoutes` function (not a method) that:
- Takes the router and database as parameters
- Handles all module initialization internally
- Manages all dependencies internally
- Registers all routes for that module

```go
// ✅ Correct: Module handles everything internally
package register

import (
	"github.com/gofiber/fiber/v2"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/otp"
	"github.com/youthcongress/authentication/v1/register/handler"
	"github.com/youthcongress/authentication/v1/register/repository"
	registerService "github.com/youthcongress/authentication/v1/register/service"
)

// SetupRoutes initializes and registers all registration-related routes
// This function handles all module initialization, dependencies, and route registration
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// Initialize OTP module (dependency)
	otpModule := otp.NewOTPModule(db.GetDB())
	
	// Initialize register module
	registerModule := NewRegisterModule(db.GetDB(), otpModule.Service)
	
	// Configure cross-module dependencies
	otpModule.SetCredentialRepository(registerModule.Repo)
	
	// Register routes
	router.Post("/register", registerModule.Handler.Register)
	router.Post("/register/validate/email", registerModule.Handler.CheckEmail)
	router.Post("/register/validate/phone", registerModule.Handler.CheckPhone)
}
```

### Route Documentation in Modules

Each module's `SetupRoutes` function should include comments documenting:
- Route paths
- HTTP methods
- Handler functions
- Request/response formats (optional)

```go
// SetupRoutes initializes and registers all registration-related routes
// Routes:
//   POST /v1/register - Register a new user
//   POST /v1/register/validate/email - Validate email uniqueness
//   POST /v1/register/validate/phone - Validate phone uniqueness
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// Initialize dependencies
	otpModule := otp.NewOTPModule(db.GetDB())
	
	// Initialize this module
	module := NewRegisterModule(db.GetDB(), otpModule.Service)
	
	// Configure dependencies
	otpModule.SetCredentialRepository(module.Repo)
	
	// Register routes
	router.Post("/register", module.Handler.Register)
	router.Post("/register/validate/email", module.Handler.CheckEmail)
	router.Post("/register/validate/phone", module.Handler.CheckPhone)
}
```

## Module Structure

### Complete Module Example

```go
package register

import (
	"github.com/gofiber/fiber/v2"
	"github.com/jackc/pgx/v5/pgxpool"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/otp"
	"github.com/youthcongress/authentication/v1/register/handler"
	"github.com/youthcongress/authentication/v1/register/repository"
	registerService "github.com/youthcongress/authentication/v1/register/service"
)

// SetupRoutes initializes and registers all registration-related routes
// Routes:
//   POST /v1/register - Register a new user
//   POST /v1/register/validate/email - Validate email uniqueness
//   POST /v1/register/validate/phone - Validate phone uniqueness
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// Initialize dependencies
	otpModule := otp.NewOTPModule(db.GetDB())
	
	// Initialize this module
	module := NewRegisterModule(db.GetDB(), otpModule.Service)
	
	// Configure dependencies
	otpModule.SetCredentialRepository(module.Repo)
	
	// Register routes
	router.Post("/register", module.Handler.Register)
	router.Post("/register/validate/email", module.Handler.CheckEmail)
	router.Post("/register/validate/phone", module.Handler.CheckPhone)
}

// RegisterModule represents the registration module
type RegisterModule struct {
	Handler *handler.RegisterHandler
	Service registerService.RegisterService
	Repo    repository.RegisterRepository
}

// NewRegisterModule creates a new register module
func NewRegisterModule(db *pgxpool.Pool, otpService otp.OTPService) *RegisterModule {
	repo := repository.NewRegisterRepository(db)
	svc := registerService.NewRegisterService(repo)
	h := handler.NewRegisterHandler(svc, otpService)

	return &RegisterModule{
		Handler: h,
		Service: svc,
		Repo:    repo,
	}
}
```

## Route Naming Conventions

### 1. RESTful Routes
- Use standard HTTP methods (GET, POST, PUT, DELETE, PATCH)
- Use plural nouns for resources
- Use nested paths for related resources

```go
// ✅ Correct: RESTful route naming
router.Post("/users", m.Handler.CreateUser)
router.Get("/users/:id", m.Handler.GetUser)
router.Put("/users/:id", m.Handler.UpdateUser)
router.Delete("/users/:id", m.Handler.DeleteUser)
```

### 2. Action-based Routes
- Use verbs for actions that don't map to CRUD operations
- Use clear, descriptive action names

```go
// ✅ Correct: Action-based routes
router.Post("/users/:id/activate", m.Handler.ActivateUser)
router.Post("/users/:id/deactivate", m.Handler.DeactivateUser)
router.Post("/users/:id/reset-password", m.Handler.ResetPassword)
```

### 3. Validation Routes
- Use `/validate` prefix for validation endpoints
- Specify what is being validated

```go
// ✅ Correct: Validation routes
router.Post("/register/validate/email", m.Handler.CheckEmail)
router.Post("/register/validate/phone", m.Handler.CheckPhone)
router.Post("/register/validate/username", m.Handler.CheckUsername)
```

## Route Grouping

### Version-based Grouping
Routes should be grouped by API version:

```go
func SetupRoutes(app *fiber.App, db *connection.Database) {
	// API Version 1
	v1 := app.Group("/v1")
	registerModule.SetupRoutes(v1)
	loginModule.SetupRoutes(v1)
	
	// API Version 2 (future)
	v2 := app.Group("/v2")
	// v2 routes...
}
```

### Feature-based Grouping
Within a version, routes can be further grouped by feature:

```go
func (m *RegisterModule) SetupRoutes(router fiber.Router) {
	// Registration group
	register := router.Group("/register")
	register.Post("/", m.Handler.Register)
	
	// Validation group
	validate := router.Group("/register/validate")
	validate.Post("/email", m.Handler.CheckEmail)
	validate.Post("/phone", m.Handler.CheckPhone)
}
```

## Middleware Application

### Global Middleware
Apply middleware at the server level:

```go
func SetupRoutes(app *fiber.App, db *connection.Database) {
	// Global middleware
	app.Use(logger.New())
	app.Use(cors.New())
	app.Use(recover.New())
	
	v1 := app.Group("/v1")
	// ... route setup
}
```

### Route-specific Middleware
Apply middleware at the module level:

```go
func (m *RegisterModule) SetupRoutes(router fiber.Router) {
	// Apply authentication middleware to all routes
	router.Use(middleware.Authenticate())
	
	router.Post("/register", m.Handler.Register)
	router.Post("/register/validate/email", m.Handler.CheckEmail)
}
```

## Best Practices

### 1. Keep Server File Clean
- ✅ Only module initialization and route registration
- ❌ No route path definitions
- ❌ No handler method calls in route definitions

### 2. Document Routes in Modules
- ✅ Document all routes in `SetupRoutes` method
- ✅ Include request/response types
- ✅ Include route purpose

### 3. Use Consistent Naming
- ✅ Use module name in route paths (e.g., `/register`, `/login`)
- ✅ Use consistent HTTP methods
- ✅ Use clear, descriptive paths

### 4. Handle Dependencies
- ✅ Set up module dependencies before route registration
- ✅ Pass required services to modules
- ✅ Configure cross-module dependencies

### 5. Error Handling
- ✅ Handle route registration errors
- ✅ Log route setup for debugging
- ✅ Validate route configurations

## Example: Complete Implementation

### Server File (`server/routes.go`)
```go
package server

import (
	"github.com/gofiber/fiber/v2"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/login"
	"github.com/youthcongress/authentication/v1/otp"
	passwordreset "github.com/youthcongress/authentication/v1/password-reset"
	"github.com/youthcongress/authentication/v1/register"
)

// SetupRoutes configures all application routes
// This file acts as an index of all route modules
func SetupRoutes(app *fiber.App, db *connection.Database) {
	v1 := app.Group("/v1")

	// Route modules index
	register.SetupRoutes(v1, db)
	login.SetupRoutes(v1, db)
	otp.SetupRoutes(v1, db)
	passwordreset.SetupRoutes(v1, db)
}
```

### Module File (`v1/register/register.module.go`)
```go
package register

import (
	"github.com/gofiber/fiber/v2"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/otp"
	// ... other imports
)

// SetupRoutes initializes and registers all registration-related routes
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// Initialize dependencies
	otpModule := otp.NewOTPModule(db.GetDB())
	
	// Initialize this module
	module := NewRegisterModule(db.GetDB(), otpModule.Service)
	
	// Configure dependencies
	otpModule.SetCredentialRepository(module.Repo)
	
	// Register routes
	router.Post("/register", module.Handler.Register)
	router.Post("/register/validate/email", module.Handler.CheckEmail)
	router.Post("/register/validate/phone", module.Handler.CheckPhone)
}
```

## Summary

- **Routes File**: Simple index of module setup calls, one line per module
- **Module Files**: Handle all initialization, dependencies, and route registration
- **Self-contained**: Each module manages its own setup completely
- **Clean Separation**: Routes file knows nothing about module internals
- **Maintainability**: Easy to add/modify modules without touching routes file

## Related Documentation

For more detailed setup instructions and examples, see:
- `server-and-routes-setup.md` - Detailed setup guide with complete examples

