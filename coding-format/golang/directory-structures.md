# Go Backend - Directory Structure Standards

## Overview
This document defines the scalable directory structure standards for Go backend projects using GoFiber framework. The structure emphasizes modularity, maintainability, and scalability for enterprise applications.

## Core Principles

### 1. Modular Architecture
- **Module-based organization**: Each feature is a self-contained module
- **Clear boundaries**: Well-defined interfaces between modules
- **Independent development**: Teams can work on modules independently
- **Easy testing**: Each module can be tested in isolation

### 2. Scalability Considerations
- **Horizontal scaling**: Easy to add new modules
- **Vertical scaling**: Easy to extend existing modules
- **Team scaling**: Multiple teams can work simultaneously
- **Feature scaling**: New features don't break existing ones

### 3. Separation of Concerns
- **Single Responsibility**: Each directory has one purpose
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Interface Segregation**: Small, focused interfaces
- **Composition over Inheritance**: Use composition for relationships

## Complete Directory Structure

```
project/
├── v1/                                    # API version 1
│   ├── dashboard/                         # Dashboard module
│   │   ├── access/                        # Access sub-module
│   │   │   ├── dto/                       # Data Transfer Objects
│   │   │   │   ├── access.dto.go
│   │   │   │   └── index.go
│   │   │   ├── handler/                   # HTTP Controllers
│   │   │   │   ├── access.handler.go
│   │   │   │   └── index.go
│   │   │   ├── service/                   # Business Logic
│   │   │   │   ├── access.service.go
│   │   │   │   └── index.go
│   │   │   ├── repository/                 # Data Access Layer
│   │   │   │   ├── access.repository.go
│   │   │   │   └── index.go
│   │   │   ├── models/                    # Data Structures
│   │   │   │   ├── access.models.go
│   │   │   │   └── index.go
│   │   │   └── access.module.go           # Module configuration
│   │   ├── assets/                        # Assets sub-module
│   │   │   ├── dto/
│   │   │   │   ├── asset.dto.go
│   │   │   │   ├── category.dto.go
│   │   │   │   └── index.go
│   │   │   ├── handler/
│   │   │   │   ├── asset.handler.go
│   │   │   │   ├── category.handler.go
│   │   │   │   └── index.go
│   │   │   ├── service/
│   │   │   │   ├── asset.service.go
│   │   │   │   ├── category.service.go
│   │   │   │   └── index.go
│   │   │   ├── repository/
│   │   │   │   ├── asset.repository.go
│   │   │   │   ├── category.repository.go
│   │   │   │   └── index.go
│   │   │   ├── models/
│   │   │   │   ├── asset.models.go
│   │   │   │   ├── category.models.go
│   │   │   │   └── index.go
│   │   │   └── assets.module.go
│   │   ├── employees/                     # Employees sub-module
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── employees.module.go
│   │   ├── stats/                         # Statistics sub-module
│   │   │   ├── dashboard/                 # Dashboard statistics
│   │   │   │   ├── unified/               # Unified dashboard
│   │   │   │   │   ├── dto/
│   │   │   │   │   ├── handler/
│   │   │   │   │   ├── service/
│   │   │   │   │   ├── repository/
│   │   │   │   │   ├── models/
│   │   │   │   │   └── unified.module.go
│   │   │   │   └── summary/               # Summary statistics
│   │   │   │       ├── dto/
│   │   │   │       ├── handler/
│   │   │   │       ├── service/
│   │   │   │       ├── repository/
│   │   │   │       ├── models/
│   │   │   │       └── summary.module.go
│   │   │   └── stats.module.go
│   │   └── dashboard.module.go            # Main dashboard module
│   ├── auth/                             # Authentication module
│   │   ├── login/                        # Login sub-module
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── login.module.go
│   │   ├── register/                     # Registration sub-module
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── register.module.go
│   │   ├── password/                     # Password management
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── password.module.go
│   │   └── auth.module.go
│   ├── user/                             # User management module
│   │   ├── profile/                      # User profile
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── profile.module.go
│   │   ├── roles/                        # User roles
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── roles.module.go
│   │   └── user.module.go
│   ├── organization/                     # Organization module
│   │   ├── company/                      # Company management
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── company.module.go
│   │   ├── branches/                     # Branch management
│   │   │   ├── dto/
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── models/
│   │   │   └── branches.module.go
│   │   └── organization.module.go
│   └── shared/                           # Shared utilities
│       ├── dto/                          # Common DTOs
│       │   ├── common.dto.go
│       │   ├── pagination.dto.go
│       │   └── index.go
│       ├── models/                       # Common models
│       │   ├── base.models.go
│       │   ├── audit.models.go
│       │   └── index.go
│       ├── utils/                        # Utility functions
│       │   ├── validation.utils.go
│       │   ├── encryption.utils.go
│       │   └── index.go
│       └── middleware/                   # Common middleware
│           ├── auth.middleware.go
│           ├── logging.middleware.go
│           └── index.go
├── connection/                           # Database connections
│   ├── postgres/
│   │   ├── connection.go
│   │   ├── migrations/
│   │   └── index.go
│   ├── redis/
│   │   ├── connection.go
│   │   └── index.go
│   └── index.go
├── docs/                                # Documentation
│   ├── api/
│   ├── architecture/
│   └── deployment/
├── scripts/                             # Utility scripts
│   ├── migration/
│   ├── seed/
│   └── backup/
├── server/                             # Server configuration
│   ├── config/
│   ├── routes/
│   └── middleware/
├── go.mod                              # Go module file
├── go.sum                              # Go module checksums
├── main.go                             # Application entry point
└── README.md                           # Project documentation
```

## Directory Structure Standards

### 1. Module Organization
**Pattern**: `v1/{module}/{sub-module}/`

```go
// ✅ Correct: Clear module hierarchy
v1/
├── dashboard/
│   ├── assets/
│   ├── employees/
│   └── stats/
├── auth/
│   ├── login/
│   ├── register/
│   └── password/
└── user/
    ├── profile/
    └── roles/

// ❌ Incorrect: Flat structure
v1/
├── assets/
├── employees/
├── login/
├── register/
└── profile/
```

### 2. Layer Organization
**Pattern**: `{module}/{layer}/`

```go
// ✅ Correct: Consistent layer structure
assets/
├── dto/           # Data Transfer Objects
├── handler/       # HTTP Controllers
├── service/       # Business Logic
├── repository/    # Data Access
├── models/        # Data Structures
└── assets.module.go

// ❌ Incorrect: Mixed responsibilities
assets/
├── controllers/
├── business/
├── data/
└── types/
```

### 3. File Naming Conventions

#### 3.1. Layer Files
```go
// ✅ Correct: Descriptive layer files
asset.handler.go
asset.service.go
asset.repository.go
asset.models.go
asset.dto.go

// ❌ Incorrect: Generic names
handler.go
service.go
repository.go
models.go
dto.go
```

#### 3.2. Module Files
```go
// ✅ Correct: Module configuration files
assets.module.go
auth.module.go
dashboard.module.go

// ❌ Incorrect: Generic names
module.go
config.go
init.go
```

#### 3.3. Index Files
```go
// ✅ Correct: Clean exports
// dto/index.go
export * from './asset.dto';
export * from './category.dto';

// handler/index.go
export * from './asset.handler';
export * from './category.handler';

// ❌ Incorrect: No index files
// Direct imports from specific files
```

### 4. Shared Resources Organization

#### 4.1. Common DTOs
```go
// shared/dto/common.dto.go
type PaginationRequest struct {
    Page  int `json:"page" validate:"min=1"`
    Limit int `json:"limit" validate:"min=1,max=100"`
}

type PaginationResponse struct {
    Data       interface{} `json:"data"`
    Total      int64       `json:"total"`
    Page       int         `json:"page"`
    Limit      int         `json:"limit"`
    TotalPages int         `json:"total_pages"`
}
```

#### 4.2. Base Models
```go
// shared/models/base.models.go
type BaseModel struct {
    CreatedAt time.Time  `db:"created_at" json:"created_at"`
    UpdatedAt *time.Time `db:"updated_at" json:"updated_at"`
    CreatedBy string     `db:"created_by" json:"created_by"`
    UpdatedBy *string    `db:"updated_by" json:"updated_by"`
}

type AuditModel struct {
    BaseModel
    DeletedAt *time.Time `db:"deleted_at" json:"deleted_at"`
    DeletedBy *string    `db:"deleted_by" json:"deleted_by"`
}
```

#### 4.3. Common Utilities
```go
// shared/utils/validation.utils.go
func ValidateEmail(email string) bool {
    // Email validation logic
}

func ValidateUUID(id string) bool {
    // UUID validation logic
}

// shared/utils/encryption.utils.go
func HashPassword(password string) (string, error) {
    // Password hashing logic
}

func VerifyPassword(hashedPassword, password string) bool {
    // Password verification logic
}
```

## Scalability Patterns

### 1. Module Independence
```go
// ✅ Correct: Independent modules
v1/
├── dashboard/          # Can be developed independently
│   ├── assets/
│   ├── employees/
│   └── stats/
├── auth/              # Can be developed independently
│   ├── login/
│   ├── register/
│   └── password/
└── user/              # Can be developed independently
    ├── profile/
    └── roles/

// ❌ Incorrect: Tightly coupled modules
v1/
├── dashboard/
│   ├── assets/
│   ├── employees/
│   ├── auth/          # Auth mixed with dashboard
│   └── user/          # User mixed with dashboard
```

### 2. Sub-module Scalability
```go
// ✅ Correct: Easy to add new sub-modules
dashboard/
├── assets/            # Existing
├── employees/         # Existing
├── stats/             # Existing
├── reports/           # New sub-module
└── analytics/         # New sub-module

// ❌ Incorrect: Hard to extend
dashboard/
├── assets/
├── employees/
└── stats/
    ├── reports/       # Reports mixed with stats
    └── analytics/     # Analytics mixed with stats
```

### 3. Layer Scalability
```go
// ✅ Correct: Easy to add new layers
assets/
├── dto/               # Existing
├── handler/           # Existing
├── service/           # Existing
├── repository/        # Existing
├── models/            # Existing
├── cache/             # New layer
└── events/            # New layer

// ❌ Incorrect: Mixed responsibilities
assets/
├── dto/
├── handler/
├── service/
├── repository/
├── models/
├── cache/             # Cache mixed with other concerns
└── events/            # Events mixed with other concerns
```

## Module Configuration Standards

### 1. Module Definition
```go
// assets.module.go
package assets

import (
    "github.com/your-project/v1/dashboard/assets/dto"
    "github.com/your-project/v1/dashboard/assets/handler"
    "github.com/your-project/v1/dashboard/assets/service"
    "github.com/your-project/v1/dashboard/assets/repository"
    "github.com/your-project/v1/dashboard/assets/models"
)

type AssetsModule struct {
    AssetHandler    *handler.AssetHandler
    CategoryHandler *handler.CategoryHandler
    AssetService    service.AssetService
    CategoryService service.CategoryService
    AssetRepo       repository.AssetRepository
    CategoryRepo    repository.CategoryRepository
}

func NewAssetsModule() *AssetsModule {
    // Initialize dependencies
    assetRepo := repository.NewAssetRepository()
    categoryRepo := repository.NewCategoryRepository()
    
    assetService := service.NewAssetService(assetRepo)
    categoryService := service.NewCategoryService(categoryRepo)
    
    assetHandler := handler.NewAssetHandler(assetService)
    categoryHandler := handler.NewCategoryHandler(categoryService)
    
    return &AssetsModule{
        AssetHandler:    assetHandler,
        CategoryHandler: categoryHandler,
        AssetService:    assetService,
        CategoryService: categoryService,
        AssetRepo:       assetRepo,
        CategoryRepo:    categoryRepo,
    }
}
```

### 2. Module Registration
```go
// dashboard.module.go
package dashboard

import (
    "github.com/your-project/v1/dashboard/assets"
    "github.com/your-project/v1/dashboard/employees"
    "github.com/your-project/v1/dashboard/stats"
)

type DashboardModule struct {
    AssetsModule    *assets.AssetsModule
    EmployeesModule  *employees.EmployeesModule
    StatsModule      *stats.StatsModule
}

func NewDashboardModule() *DashboardModule {
    return &DashboardModule{
        AssetsModule:   assets.NewAssetsModule(),
        EmployeesModule: employees.NewEmployeesModule(),
        StatsModule:    stats.NewStatsModule(),
    }
}
```

## Import Standards

### 1. Module Imports
```go
// ✅ Correct: Clean module imports
import (
    "github.com/your-project/v1/dashboard/assets"
    "github.com/your-project/v1/dashboard/employees"
    "github.com/your-project/v1/auth"
    "github.com/your-project/v1/user"
)

// ❌ Incorrect: Direct file imports
import (
    "github.com/your-project/v1/dashboard/assets/handler"
    "github.com/your-project/v1/dashboard/assets/service"
    "github.com/your-project/v1/dashboard/employees/handler"
    "github.com/your-project/v1/auth/login/handler"
)
```

### 2. Shared Imports
```go
// ✅ Correct: Shared utilities
import (
    "github.com/your-project/v1/shared/dto"
    "github.com/your-project/v1/shared/models"
    "github.com/your-project/v1/shared/utils"
)

// ❌ Incorrect: Direct shared file imports
import (
    "github.com/your-project/v1/shared/dto/common.dto"
    "github.com/your-project/v1/shared/models/base.models"
    "github.com/your-project/v1/shared/utils/validation.utils"
)
```

## Testing Structure

### 1. Test Organization
```
v1/
├── dashboard/
│   ├── assets/
│   │   ├── dto/
│   │   │   └── asset.dto_test.go
│   │   ├── handler/
│   │   │   └── asset.handler_test.go
│   │   ├── service/
│   │   │   └── asset.service_test.go
│   │   ├── repository/
│   │   │   └── asset.repository_test.go
│   │   └── models/
│   │       └── asset.models_test.go
│   └── ...
└── shared/
    ├── dto/
    │   └── common.dto_test.go
    └── utils/
        └── validation.utils_test.go
```

### 2. Test File Naming
```go
// ✅ Correct: Test file naming
asset.handler_test.go
asset.service_test.go
asset.repository_test.go
asset.models_test.go

// ❌ Incorrect: Generic test names
handler_test.go
service_test.go
repository_test.go
models_test.go
```

## Migration Strategy

### 1. Phase 1: Foundation
- Create base directory structure
- Implement shared utilities
- Set up module configuration

### 2. Phase 2: Core Modules
- Implement authentication module
- Implement user management module
- Implement dashboard module

### 3. Phase 3: Feature Modules
- Add asset management
- Add employee management
- Add statistics module

### 4. Phase 4: Advanced Features
- Add reporting module
- Add analytics module
- Add integration modules

## Best Practices Summary

### 1. Directory Structure
- Use consistent naming conventions
- Maintain clear module boundaries
- Keep related functionality together
- Separate concerns properly

### 2. Scalability
- Design for horizontal scaling
- Plan for team growth
- Consider feature expansion
- Maintain backward compatibility

### 3. Maintainability
- Use clear import paths
- Maintain consistent patterns
- Document module interfaces
- Keep dependencies minimal

### 4. Testing
- Test each layer independently
- Use consistent test structure
- Maintain high test coverage
- Test module integration

This directory structure ensures scalability, maintainability, and team productivity for your Go backend project.
