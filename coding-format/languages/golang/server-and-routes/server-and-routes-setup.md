# Go Backend - Server and Routes Setup Guide

## Overview
This document provides detailed guidelines for setting up server and routes in Go backend projects using GoFiber framework. The architecture emphasizes minimal server configuration with module-based route management.

## Core Principles

### 1. Minimal Server Configuration
- **Routes File**: Acts as an index of route modules only
- **Module Independence**: Each module handles its own initialization and dependencies
- **Clean Separation**: Server file only lists modules, no implementation details
- **Self-contained Modules**: Modules manage their own dependencies and route setup

### 2. Module Responsibility
- **Initialization**: Each module initializes its own dependencies
- **Dependency Management**: Modules handle cross-module dependencies internally
- **Route Registration**: Modules register their own routes
- **Configuration**: Modules configure their own settings

## Server Routes File Structure

### Routes File (`server/routes.go`)

The routes file should be a simple index of all route modules. It should:
- Only list module names
- Not contain any initialization logic
- Not contain any dependency setup
- Not contain any route path definitions
- Be easy to scan and understand

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

## Module Setup Function Pattern

### Module Setup Function

Each module should export a `SetupRoutes` function that:
- Takes the router and database as parameters
- Handles all module initialization internally
- Manages all dependencies internally
- Registers all routes for that module
- Returns nothing (void function)

```go
// ✅ Correct: Module handles everything internally
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

// RegisterModule represents the registration module with all dependencies
type RegisterModule struct {
	Handler *handler.RegisterHandler
	Service registerService.RegisterService
	Repo    repository.RegisterRepository
}

// NewRegisterModule creates a new register module with all dependencies
func NewRegisterModule(db *pgxpool.Pool, otpService otp.OTPService) *RegisterModule {
	// Initialize repository layer
	repo := repository.NewRegisterRepository(db)

	// Initialize service layer
	svc := registerService.NewRegisterService(repo)

	// Initialize handler layer
	h := handler.NewRegisterHandler(svc, otpService)

	return &RegisterModule{
		Handler: h,
		Service: svc,
		Repo:    repo,
	}
}
```

## Module Structure

### Complete Module Example

Each module directory should contain:

```
v1/
├── register/
│   ├── handler/
│   │   └── register.handler.go
│   ├── service/
│   │   └── register.service.go
│   ├── repository/
│   │   └── register.repository.go
│   ├── dto/
│   │   └── register.dto.go
│   ├── models/
│   │   └── user.models.go
│   └── register.module.go      # Contains SetupRoutes function
```

### Module File Structure

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

## Simple Module Example (No Dependencies)

For modules that don't have dependencies on other modules:

```go
package login

import (
	"github.com/gofiber/fiber/v2"
	"github.com/jackc/pgx/v5/pgxpool"
	"github.com/youthcongress/authentication/connection"
	"github.com/youthcongress/authentication/v1/login/handler"
	"github.com/youthcongress/authentication/v1/login/repository"
	loginService "github.com/youthcongress/authentication/v1/login/service"
)

// SetupRoutes initializes and registers all login-related routes
// Routes:
//   POST /v1/login - Authenticate user and login
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// Initialize module
	module := NewLoginModule(db.GetDB())
	
	// Register routes
	router.Post("/login", module.Handler.Login)
}

// LoginModule represents the login module
type LoginModule struct {
	Handler *handler.LoginHandler
	Service loginService.LoginService
	Repo    repository.LoginRepository
}

// NewLoginModule creates a new login module
func NewLoginModule(db *pgxpool.Pool) *LoginModule {
	repo := repository.NewLoginRepository(db)
	svc := loginService.NewLoginService(repo)
	h := handler.NewLoginHandler(svc)

	return &LoginModule{
		Handler: h,
		Service: svc,
		Repo:    repo,
	}
}
```

## Dependency Management

### Handling Module Dependencies

When a module depends on another module:

1. **Import the dependency module** in the module file
2. **Initialize the dependency** inside `SetupRoutes`
3. **Pass dependencies** to module constructor
4. **Configure cross-module settings** if needed

```go
// Example: Register module depends on OTP module
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// 1. Initialize dependency module
	otpModule := otp.NewOTPModule(db.GetDB())
	
	// 2. Initialize this module with dependency
	registerModule := NewRegisterModule(db.GetDB(), otpModule.Service)
	
	// 3. Configure cross-module dependencies
	otpModule.SetCredentialRepository(registerModule.Repo)
	
	// 4. Register routes
	router.Post("/register", registerModule.Handler.Register)
	// ... more routes
}
```

### Shared Modules

For modules that are used by multiple other modules (like OTP):

```go
package otp

// SetupRoutes initializes and registers all OTP-related routes
// Note: OTP module is also used as a dependency by other modules
func SetupRoutes(router fiber.Router, db *connection.Database) {
	module := NewOTPModule(db.GetDB())
	
	router.Post("/otp/generate", module.Handler.GenerateOTP)
	router.Post("/otp/verify", module.Handler.VerifyOTP)
}

// NewOTPModule creates a new OTP module (exported for use by other modules)
func NewOTPModule(db *pgxpool.Pool) *OTPModule {
	// ... initialization
}
```

## Route Documentation

### Inline Route Documentation

Each `SetupRoutes` function should document:
- All route paths
- HTTP methods
- Purpose of each route

```go
// SetupRoutes initializes and registers all registration-related routes
// Routes:
//   POST /v1/register - Register a new user
//     Request: RegisterRequest
//     Response: RegisterResponse
//   POST /v1/register/validate/email - Validate email uniqueness
//     Request: ValidateEmailRequest
//     Response: ValidateResponse
//   POST /v1/register/validate/phone - Validate phone uniqueness
//     Request: ValidatePhoneRequest
//     Response: ValidateResponse
func SetupRoutes(router fiber.Router, db *connection.Database) {
	// ... implementation
}
```

## Best Practices

### 1. Keep Routes File Minimal
- ✅ Only list module setup calls
- ✅ One line per module
- ❌ No initialization logic
- ❌ No dependency management
- ❌ No route path definitions

### 2. Encapsulate in Modules
- ✅ All initialization in module's `SetupRoutes`
- ✅ All dependency management in module
- ✅ All route registration in module
- ✅ Module is self-contained

### 3. Clear Function Signatures
- ✅ `SetupRoutes(router fiber.Router, db *connection.Database)`
- ✅ Consistent across all modules
- ✅ Easy to understand parameters

### 4. Document Routes
- ✅ Document all routes in `SetupRoutes` function
- ✅ Include HTTP methods and paths
- ✅ Include request/response types if helpful

### 5. Handle Dependencies Internally
- ✅ Initialize dependencies inside `SetupRoutes`
- ✅ Configure cross-module dependencies inside module
- ✅ Don't expose internal module structure to routes file

## Complete Example

### Server Routes File (`server/routes.go`)
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

## Summary

- **Routes File**: Simple index of module setup calls, one line per module
- **Module Files**: Handle all initialization, dependencies, and route registration
- **Self-contained**: Each module manages its own setup completely
- **Clean Separation**: Routes file knows nothing about module internals
- **Maintainable**: Easy to add/modify modules without touching routes file

