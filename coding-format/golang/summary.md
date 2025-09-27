# Go Backend Coding Format - Summary

## Overview
This document defines the industry-standard coding format for Go backend projects using GoFiber framework. The format emphasizes clean MVC architecture, scalability, maintainability, and follows Go best practices with a simple, organized directory structure.

## Core Principles

### 1. MVC Architecture
- **Model**: Data structures and business entities
- **View**: API responses and data presentation  
- **Controller**: HTTP request/response handling

### 2. Layered Architecture
```
┌─────────────────┐
│    Handler      │ ← HTTP Layer (Controllers)
├─────────────────┤
│    Service      │ ← Business Logic Layer
├─────────────────┤
│   Repository    │ ← Data Access Layer
├─────────────────┤
│     Models      │ ← Data Structures
├─────────────────┤
│      DTO        │ ← Data Transfer Objects
└─────────────────┘
```

### 3. Separation of Concerns
- **Single Responsibility**: Each component has one reason to change
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Interface Segregation**: Small, focused interfaces

## Directory Structure

```
project/
├── v1/                           # API version 1
│   ├── dashboard/               # Dashboard module
│   │   ├── access/              # Access sub-module
│   │   │   ├── dto/             # Data Transfer Objects
│   │   │   │   └── access.dto.go
│   │   │   ├── handler/         # HTTP Controllers
│   │   │   │   └── access.handler.go
│   │   │   ├── service/         # Business Logic
│   │   │   │   └── access.service.go
│   │   │   ├── repository/      # Data Access Layer
│   │   │   │   └── access.repository.go
│   │   │   ├── models/          # Data Structures
│   │   │   │   └── access.models.go
│   │   │   └── access.module.go # Module configuration
│   │   ├── stats/               # Statistics sub-module
│   │   │   ├── dashboard/       # Dashboard statistics
│   │   │   │   ├── unified/     # Unified dashboard
│   │   │   │   │   ├── dto/
│   │   │   │   │   ├── handler/
│   │   │   │   │   ├── service/
│   │   │   │   │   ├── repository/
│   │   │   │   │   ├── models/
│   │   │   │   │   └── unified.module.go
│   │   │   │   └── summery/     # Summary statistics
│   │   │   │       ├── dto/
│   │   │   │       ├── handler/
│   │   │   │       ├── service/
│   │   │   │       ├── repository/
│   │   │   │       ├── models/
│   │   │   │       └── summery.module.go
│   │   │   └── stats.module.go
│   │   └── dashboard.module.go
│   ├── auth/                    # Authentication module
│   │   ├── login/
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── login.module.go
│   │   └── auth.module.go
│   └── user/                    # User management module
│       ├── dto/
│       ├── handler/
│       ├── service/
│       ├── repository/
│       ├── models/
│       └── user.module.go
├── connection/                  # Database connections
├── docs/                       # Documentation
├── scripts/                    # Utility scripts
├── server/                     # Server configuration
├── go.mod                      # Go module file
├── go.sum                      # Go module checksums
├── main.go                     # Application entry point
└── README.md                   # Project documentation
```

## Key Components & Responsibilities

### 1. DTO (Data Transfer Objects)
**Location**: `{module}/dto/`
**File Naming**: `{module}.dto.go`

**Purpose**: Define data structures for API requests and responses
**Responsibilities**:
- Request/Response structures for API endpoints
- Input validation with struct tags
- JSON serialization/deserialization
- Data sanitization and transformation
- Error response formats
- API versioning support

**Key Features**:
- Validation tags (`validate:"required"`, `json:"field_name"`)
- Input sanitization
- Clear separation from internal models
- Support for nested structures
- Consistent error response format

### 2. Handler (Controllers)
**Location**: `{module}/handler/`
**File Naming**: `{module}.handler.go`

**Purpose**: Handle HTTP requests and responses
**Responsibilities**:
- HTTP request parsing and validation
- Response formatting and serialization
- Error handling and status codes
- Authentication and authorization
- Route handling and middleware integration
- Input sanitization and validation
- Logging and monitoring

**Key Features**:
- Thin layer that delegates to services
- Consistent error handling
- Request/response logging
- Authentication middleware integration
- Input validation using DTOs
- Proper HTTP status codes

### 3. Service (Business Logic)
**Location**: `{module}/service/`
**File Naming**: `{module}.service.go`

**Purpose**: Implement business logic and data processing
**Responsibilities**:
- Core business rule implementation
- Complex data processing and transformation
- Transaction management
- External service integration
- Data validation and business rules
- Error handling and logging
- Use case orchestration

**Key Features**:
- Contains core business logic
- Independent of external concerns
- Uses dependency injection
- Handles complex operations
- Manages transactions
- Integrates with repositories

### 4. Repository (Data Access)
**Location**: `{module}/repository/`
**File Naming**: `{module}.repository.go`

**Purpose**: Abstract database operations and data access
**Responsibilities**:
- Database query execution
- Data persistence and retrieval
- Query optimization and performance
- Connection management
- Data mapping between database and models
- Transaction handling
- Database-specific logic

**Key Features**:
- Abstracts database implementation
- Optimized SQL queries
- Proper error handling
- Connection pooling
- Data mapping utilities
- Transaction support

### 5. Models (Data Structures)
**Location**: `{module}/models/`
**File Naming**: `{module}.models.go`

**Purpose**: Define core business entities and data structures
**Responsibilities**:
- Business entity definitions
- Database table mappings
- Validation rules and constraints
- Business logic methods
- Entity relationships and associations
- Data integrity rules

**Key Features**:
- Represent core business concepts
- Include validation methods
- Define business rules
- Database mapping tags
- Relationship definitions
- Immutable where appropriate

**Database Schema Mapping Standards**:
- Use complete schema.table.column names in db tags
- Include proper data types and constraints
- Document foreign key relationships
- Specify nullable fields with pointers
- Include audit fields (created_at, updated_at, created_by, updated_by)

**Example Model Structure**:
```go
// User represents the user data model
type User struct {
    ID            string    `json:"id" db:"core.users.id"`
    OrganizationID string   `json:"organization_id" db:"core.users.organization_id"`
    BranchID      *string   `json:"branch_id,omitempty" db:"core.users.branch_id"`
    Name          string    `json:"name" db:"core.users.name"`
    Email         string    `json:"email" db:"core.users.email"`
    Password      string    `json:"-" db:"core.users.password"`
    IsActive      bool      `json:"is_active" db:"core.users.is_active"`
    CreatedBy     *string   `json:"created_by,omitempty" db:"core.users.created_by"`
    CreatedAt     time.Time `json:"created_at" db:"core.users.created_at"`
    UpdatedBy     *string   `json:"updated_by,omitempty" db:"core.users.updated_by"`
    UpdatedAt     time.Time `json:"updated_at" db:"core.users.updated_at"`
}
```

**Database Schema Documentation Requirements**:
- Document all table relationships
- Include constraint information
- Specify data types and lengths
- Document foreign key references
- Include index information
- Document audit trail fields

### 6. Module Configuration
**Location**: `{module}/`
**File Naming**: `{module}.module.go`

**Purpose**: Module configuration and dependency injection
**Responsibilities**:
- Dependency injection setup
- Module initialization
- Route registration
- Service configuration
- Controller setup
- Middleware configuration

**Key Features**:
- Centralized module configuration
- Dependency injection
- Route definitions
- Service initialization
- Middleware setup

## Naming Conventions

### Directory Structure
- Use snake_case for directory names: `dashboard/`, `user_management/`
- Group related modules: `dashboard/access/`, `dashboard/stats/`
- Keep module names descriptive and clear

### File Naming
- Use snake_case for file names: `access.dto.go`, `user.handler.go`
- Include component type: `{module}.{component}.go`
- Be consistent across all modules

### Functions
- Use PascalCase for exported functions: `GetUserData()`, `CreateUser()`
- Use camelCase for private functions: `validateInput()`, `processData()`
- Use descriptive, action-oriented names
- Follow Go naming conventions

### Variables
- Use camelCase for variables: `userID`, `organizationName`
- Use descriptive names that explain purpose
- Avoid abbreviations and acronyms
- Use meaningful names for complex operations

### Constants
- Use PascalCase for exported constants: `MaxRetries`, `DefaultTimeout`
- Use camelCase for private constants: `maxRetries`, `defaultTimeout`
- Group related constants together
- Use meaningful constant names

### Structs and Interfaces
- Use PascalCase for exported types: `User`, `UserService`
- Use descriptive names: `UserRepository`, `AuthenticationHandler`
- Include type information in names: `UserDTO`, `UserModel`

## Error Handling

### Error Types
- **Validation Errors**: Input validation failures
- **Business Logic Errors**: Domain rule violations  
- **System Errors**: Infrastructure failures
- **Authentication Errors**: Auth/authorization failures
- **Database Errors**: Data access failures

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "reason": "Invalid email format"
    }
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "request_id": "req_123456789"
}
```

### Error Handling Pattern
```go
// In Handler
func (h *UserHandler) CreateUser(c *fiber.Ctx) error {
    var req dto.CreateUserRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(400).JSON(dto.ErrorResponse{
            Success: false,
            Error: dto.ErrorDetail{
                Code:    "VALIDATION_ERROR",
                Message: "Invalid request body",
                Details: err.Error(),
            },
        })
    }
    
    result, err := h.userService.CreateUser(&req)
    if err != nil {
        return c.Status(500).JSON(dto.ErrorResponse{
            Success: false,
            Error: dto.ErrorDetail{
                Code:    "SERVICE_ERROR", 
                Message: "Failed to create user",
                Details: err.Error(),
            },
        })
    }
    
    return c.Status(201).JSON(dto.SuccessResponse{
        Success: true,
        Data:    result,
    })
}
```

## Database Standards

### Query Patterns
- Use direct SQL queries for performance
- Implement SQL injection protection with string escaping
- Use prepared statements for user input
- Include proper error handling and logging
- Optimize queries with proper indexing

### Connection Management
- Use connection pooling
- Implement proper timeout handling
- Use context for cancellation
- Handle connection failures gracefully
- Use transactions for multi-step operations

### Database Schema Standards

#### Complete Schema Mapping
All database field mappings must use complete schema.table.column names:

```go
// ✅ Correct - Complete schema.table.column mapping
type User struct {
    ID            string    `json:"id" db:"core.users.id"`
    OrganizationID string   `json:"organization_id" db:"core.users.organization_id"`
    BranchID      *string   `json:"branch_id,omitempty" db:"core.users.branch_id"`
    Name          string    `json:"name" db:"core.users.name"`
    Email         string    `json:"email" db:"core.users.email"`
    Password      string    `json:"-" db:"core.users.password"`
    IsActive      bool      `json:"is_active" db:"core.users.is_active"`
    CreatedBy     *string   `json:"created_by,omitempty" db:"core.users.created_by"`
    CreatedAt     time.Time `json:"created_at" db:"core.users.created_at"`
    UpdatedBy     *string   `json:"updated_by,omitempty" db:"core.users.updated_by"`
    UpdatedAt     time.Time `json:"updated_at" db:"core.users.updated_at"`
}

// ❌ Incorrect - Incomplete column mapping
type User struct {
    ID   string `json:"id" db:"id"`
    Name string `json:"name" db:"name"`
}
```

#### Schema Documentation Requirements
Each model must include comprehensive schema documentation:

```go
// User represents the user data model
// Schema: core.users
// Primary Key: core.users.id
// Foreign Keys: 
//   - core.users.organization_id -> core.organizations.id
//   - core.users.branch_id -> core.branches.id (nullable)
//   - core.users.created_by -> core.users.id (nullable)
//   - core.users.updated_by -> core.users.id (nullable)
// Indexes:
//   - core.users.email (unique)
//   - core.users.organization_id
//   - core.users.is_active
// Constraints:
//   - core.users.email NOT NULL UNIQUE
//   - core.users.organization_id NOT NULL
//   - core.users.name NOT NULL
type User struct {
    ID            string    `json:"id" db:"core.users.id"`                    // UUID, Primary Key
    OrganizationID string   `json:"organization_id" db:"core.users.organization_id"` // UUID, Foreign Key -> core.organizations.id
    BranchID      *string   `json:"branch_id,omitempty" db:"core.users.branch_id"`   // UUID, Foreign Key -> core.branches.id (nullable)
    Name          string    `json:"name" db:"core.users.name"`                // VARCHAR(255), NOT NULL
    Email         string    `json:"email" db:"core.users.email"`              // VARCHAR(255), NOT NULL UNIQUE
    Password      string    `json:"-" db:"core.users.password"`               // VARCHAR(255), NOT NULL (hidden from JSON)
    IsActive      bool      `json:"is_active" db:"core.users.is_active"`       // BOOLEAN, DEFAULT true
    CreatedBy     *string   `json:"created_by,omitempty" db:"core.users.created_by"` // UUID, Foreign Key -> core.users.id (nullable)
    CreatedAt     time.Time `json:"created_at" db:"core.users.created_at"`     // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    UpdatedBy     *string   `json:"updated_by,omitempty" db:"core.users.updated_by"` // UUID, Foreign Key -> core.users.id (nullable)
    UpdatedAt     time.Time `json:"updated_at" db:"core.users.updated_at"`     // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
}
```

#### Common Schema Patterns

**Core Schema Tables**:
- `core.organizations` - Organization management
- `core.branches` - Branch management  
- `core.users` - User management
- `core.positions` - Position management
- `core.access` - Access control

**Assets Schema Tables**:
- `assets.items` - Asset items
- `assets.categories` - Asset categories
- `assets.manufacturer` - Manufacturers
- `assets.model_number` - Model numbers
- `assets.tags` - Asset tags
- `assets.service_providers` - Service providers

**Vehicles Schema Tables**:
- `vehicles.bluebook` - Vehicle bluebook
- `vehicles.insurance` - Vehicle insurance
- `vehicles.pollution_sticker` - Pollution stickers

#### Audit Trail Standards
All tables must include audit trail fields:

```go
// Standard audit fields for all tables
type AuditFields struct {
    CreatedBy *string   `json:"created_by,omitempty" db:"{schema}.{table}.created_by"` // UUID, Foreign Key -> core.users.id
    CreatedAt time.Time `json:"created_at" db:"{schema}.{table}.created_at"`           // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    UpdatedBy *string   `json:"updated_by,omitempty" db:"{schema}.{table}.updated_by"` // UUID, Foreign Key -> core.users.id
    UpdatedAt time.Time `json:"updated_at" db:"{schema}.{table}.updated_at"`           // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
}
```

#### Foreign Key Documentation
All foreign key relationships must be documented:

```go
// Example with foreign key documentation
type Asset struct {
    ID             string    `json:"id" db:"assets.items.id"`                    // UUID, Primary Key
    OrganizationID string    `json:"organization_id" db:"assets.items.organization_id"` // UUID, Foreign Key -> core.organizations.id
    BranchID       string    `json:"branch_id" db:"assets.items.branch_id"`            // UUID, Foreign Key -> core.branches.id
    CategoryID     *string   `json:"category_id,omitempty" db:"assets.items.category_id"` // UUID, Foreign Key -> assets.categories.id (nullable)
    ManufacturerID *string   `json:"manufacturer_id,omitempty" db:"assets.items.manufacturer_id"` // UUID, Foreign Key -> assets.manufacturer.id (nullable)
    ModelNumberID  *string   `json:"model_number_id,omitempty" db:"assets.items.model_number_id"` // UUID, Foreign Key -> assets.model_number.id (nullable)
    // ... other fields
}
```

### Repository Pattern Example
```go
// Repository Interface
type UserRepository interface {
    CreateUser(user *models.User) error
    GetUserByID(id string) (*models.User, error)
    UpdateUser(user *models.User) error
    DeleteUser(id string) error
}

// Repository Implementation
type userRepository struct {
    db *sql.DB
}

func (r *userRepository) CreateUser(user *models.User) error {
    query := `INSERT INTO users (id, name, email, created_at) VALUES ($1, $2, $3, $4)`
    _, err := r.db.Exec(query, user.ID, user.Name, user.Email, user.CreatedAt)
    return err
}
```

## Security Standards

### Input Validation
- Validate all user inputs
- Sanitize data before processing
- Use whitelist validation where possible
- Implement rate limiting

### Authentication & Authorization
- Use JWT tokens for authentication
- Implement role-based access control
- Validate permissions at handler level
- Use secure session management

## Performance Standards

### Optimization
- Use database indexing
- Implement query optimization
- Use connection pooling
- Implement caching where appropriate

### Monitoring
- Log important operations
- Monitor response times
- Track error rates
- Implement health checks

## Testing Standards

### Test Types
- **Unit Tests**: Test individual functions
- **Integration Tests**: Test component interactions
- **End-to-End Tests**: Test complete workflows
- **Performance Tests**: Test system performance

### Test Structure
- Use table-driven tests
- Test both success and error scenarios
- Use mocks for external dependencies
- Maintain high test coverage

## Documentation Standards

### Code Documentation
- Document all exported functions
- Include usage examples
- Document complex business logic
- Keep comments up to date

### API Documentation
- Document all endpoints
- Include request/response examples
- Document error codes
- Maintain changelog

## Deployment Standards

### Environment Management
- Use environment variables for configuration
- Separate development, staging, and production
- Use secure secret management
- Implement proper logging

### Containerization
- Use Docker for containerization
- Implement multi-stage builds
- Use non-root users
- Optimize image size

## Complete Module Example

### User Management Module Structure
```
v1/user/
├── dto/
│   └── user.dto.go          # Request/Response DTOs
├── handler/
│   └── user.handler.go      # HTTP Controllers
├── service/
│   └── user.service.go      # Business Logic
├── repository/
│   └── user.repository.go   # Data Access
├── models/
│   └── user.models.go       # Data Structures
└── user.module.go           # Module Configuration
```

### DTO Example (user.dto.go)
```go
package dto

type CreateUserRequest struct {
    Name     string `json:"name" validate:"required,min=2,max=100"`
    Email    string `json:"email" validate:"required,email"`
    Password string `json:"password" validate:"required,min=8"`
}

type UserResponse struct {
    ID        string    `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

type ErrorResponse struct {
    Success bool        `json:"success"`
    Error   ErrorDetail `json:"error"`
}

type ErrorDetail struct {
    Code    string      `json:"code"`
    Message string      `json:"message"`
    Details interface{} `json:"details,omitempty"`
}
```

### Handler Example (user.handler.go)
```go
package handler

type UserHandler struct {
    userService service.UserService
}

func NewUserHandler(userService service.UserService) *UserHandler {
    return &UserHandler{userService: userService}
}

func (h *UserHandler) CreateUser(c *fiber.Ctx) error {
    var req dto.CreateUserRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(400).JSON(dto.ErrorResponse{
            Success: false,
            Error: dto.ErrorDetail{
                Code:    "VALIDATION_ERROR",
                Message: "Invalid request body",
            },
        })
    }
    
    result, err := h.userService.CreateUser(&req)
    if err != nil {
        return c.Status(500).JSON(dto.ErrorResponse{
            Success: false,
            Error: dto.ErrorDetail{
                Code:    "SERVICE_ERROR",
                Message: "Failed to create user",
            },
        })
    }
    
    return c.Status(201).JSON(dto.SuccessResponse{
        Success: true,
        Data:    result,
    })
}
```

### Service Example (user.service.go)
```go
package service

type UserService interface {
    CreateUser(req *dto.CreateUserRequest) (*dto.UserResponse, error)
    GetUserByID(id string) (*dto.UserResponse, error)
}

type userService struct {
    userRepo repository.UserRepository
}

func NewUserService(userRepo repository.UserRepository) UserService {
    return &userService{userRepo: userRepo}
}

func (s *userService) CreateUser(req *dto.CreateUserRequest) (*dto.UserResponse, error) {
    // Business logic validation
    if err := s.validateUserData(req); err != nil {
        return nil, err
    }
    
    // Create user model
    user := &models.User{
        ID:        uuid.New().String(),
        Name:      req.Name,
        Email:     req.Email,
        Password:  s.hashPassword(req.Password),
        CreatedAt: time.Now(),
    }
    
    // Save to repository
    if err := s.userRepo.CreateUser(user); err != nil {
        return nil, err
    }
    
    // Return response DTO
    return &dto.UserResponse{
        ID:        user.ID,
        Name:      user.Name,
        Email:     user.Email,
        CreatedAt: user.CreatedAt,
        UpdatedAt: user.UpdatedAt,
    }, nil
}
```

### Repository Example (user.repository.go)
```go
package repository

type UserRepository interface {
    CreateUser(user *models.User) error
    GetUserByID(id string) (*models.User, error)
}

type userRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) CreateUser(user *models.User) error {
    query := `INSERT INTO users (id, name, email, password, created_at, updated_at) 
              VALUES ($1, $2, $3, $4, $5, $6)`
    _, err := r.db.Exec(query, user.ID, user.Name, user.Email, user.Password, user.CreatedAt, user.UpdatedAt)
    return err
}
```

### Models Example (user.models.go)
```go
package models

import (
    "errors"
    "time"
)

// User represents the user data model
// Schema: core.users
// Primary Key: core.users.id
// Foreign Keys: 
//   - core.users.organization_id -> core.organizations.id
//   - core.users.branch_id -> core.branches.id (nullable)
//   - core.users.created_by -> core.users.id (nullable)
//   - core.users.updated_by -> core.users.id (nullable)
// Indexes:
//   - core.users.email (unique)
//   - core.users.organization_id
//   - core.users.is_active
// Constraints:
//   - core.users.email NOT NULL UNIQUE
//   - core.users.organization_id NOT NULL
//   - core.users.name NOT NULL
type User struct {
    ID            string    `json:"id" db:"core.users.id"`                    // UUID, Primary Key
    OrganizationID string   `json:"organization_id" db:"core.users.organization_id"` // UUID, Foreign Key -> core.organizations.id
    BranchID      *string   `json:"branch_id,omitempty" db:"core.users.branch_id"`   // UUID, Foreign Key -> core.branches.id (nullable)
    Name          string    `json:"name" db:"core.users.name"`                // VARCHAR(255), NOT NULL
    Email         string    `json:"email" db:"core.users.email"`              // VARCHAR(255), NOT NULL UNIQUE
    Password      string    `json:"-" db:"core.users.password"`               // VARCHAR(255), NOT NULL (hidden from JSON)
    IsActive      bool      `json:"is_active" db:"core.users.is_active"`       // BOOLEAN, DEFAULT true
    CreatedBy     *string   `json:"created_by,omitempty" db:"core.users.created_by"` // UUID, Foreign Key -> core.users.id (nullable)
    CreatedAt     time.Time `json:"created_at" db:"core.users.created_at"`     // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
    UpdatedBy     *string   `json:"updated_by,omitempty" db:"core.users.updated_by"` // UUID, Foreign Key -> core.users.id (nullable)
    UpdatedAt     time.Time `json:"updated_at" db:"core.users.updated_at"`     // TIMESTAMP, DEFAULT CURRENT_TIMESTAMP
}

func (u *User) Validate() error {
    if u.Name == "" {
        return errors.New("name is required")
    }
    if u.Email == "" {
        return errors.New("email is required")
    }
    if u.OrganizationID == "" {
        return errors.New("organization_id is required")
    }
    return nil
}
```

### Module Configuration (user.module.go)
```go
package user

type UserModule struct {
    handler *handler.UserHandler
    service service.UserService
    repo    repository.UserRepository
}

func NewUserModule(db *sql.DB) *UserModule {
    // Initialize repository
    userRepo := repository.NewUserRepository(db)
    
    // Initialize service
    userService := service.NewUserService(userRepo)
    
    // Initialize handler
    userHandler := handler.NewUserHandler(userService)
    
    return &UserModule{
        handler: userHandler,
        service: userService,
        repo:    userRepo,
    }
}

func (m *UserModule) GetRoutes() []fiber.Route {
    return []fiber.Route{
        {Method: "POST", Path: "/users", Handler: m.handler.CreateUser},
        {Method: "GET", Path: "/users/:id", Handler: m.handler.GetUserByID},
    }
}
```

## Best Practices

### Code Quality
- Write clean, readable code with meaningful names
- Keep functions small and focused (max 20-30 lines)
- Follow Go idioms and conventions
- Use consistent formatting with gofmt

### Maintainability
- Use consistent error handling patterns
- Write comprehensive tests for all layers
- Keep dependencies up to date
- Document complex business logic

### Scalability
- Design for horizontal scaling with stateless services
- Implement proper caching strategies
- Use connection pooling for database operations
- Monitor resource usage and performance

### Security
- Validate all inputs at handler level
- Use parameterized queries to prevent SQL injection
- Implement proper authentication and authorization
- Sanitize data before processing

### Performance
- Use direct SQL queries for complex operations
- Implement proper database indexing
- Use goroutines for concurrent operations
- Monitor and optimize slow queries

This coding format ensures consistency, maintainability, and scalability across all Go backend projects while following industry best practices and Go conventions. The modular structure allows for easy testing, debugging, and future enhancements.
