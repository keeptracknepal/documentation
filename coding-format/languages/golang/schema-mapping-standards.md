# Go Backend - Schema Mapping Standards

## Overview
This document defines the standards for mapping database schemas to Go structs, ensuring consistency, type safety, and maintainability across the backend codebase.

## Core Principles

### 1. Complete Schema Mapping
- Use complete `schema.table.column` names in database tags
- Include proper data types and constraints
- Document foreign key relationships
- Specify nullable fields with pointers
- Include audit fields for tracking

### 2. Type Safety
- Use appropriate Go types for database columns
- Handle nullable fields with pointers
- Use custom types for enums and status fields
- Implement proper validation

### 3. Consistency
- Standardize naming conventions
- Use consistent tag formats
- Follow established patterns for relationships
- Maintain audit trail standards

## Database Schema Mapping Standards

### 1. Basic Struct Mapping

```go
// ✅ Correct: Complete schema mapping with proper types
type User struct {
    ID          string     `db:"core.users.id"            json:"core.users.id"`
    Username    string     `db:"core.users.username"      json:"core.users.username"`
    Email       string     `db:"core.users.email"         json:"core.users.email"`
    Status      string     `db:"core.users.status"        json:"core.users.status"`
    CreatedAt   time.Time  `db:"core.users.created_at"    json:"core.users.created_at"`
    UpdatedAt   *time.Time `db:"core.users.updated_at"    json:"core.users.updated_at"`
    CreatedBy   string     `db:"core.users.created_by"    json:"core.users.created_by"`
    UpdatedBy   *string    `db:"core.users.updated_by"    json:"core.users.updated_by"`
}

// ❌ Incorrect: Missing schema, incomplete mapping
type User struct {
    ID       string `db:"id" json:"id"`
    Username string `db:"username" json:"username"`
}
```

### 2. Nullable Fields Handling

```go
// ✅ Correct: Use pointers for nullable fields
type Asset struct {
    ID              string     `db:"core.assets.id"               json:"core.assets.id"`
    Name            string     `db:"core.assets.name"             json:"core.assets.name"`
    ModelNumber     *string    `db:"core.assets.model_number"     json:"core.assets.model_number"`
    Manufacturer    *string    `db:"core.assets.manufacturer"     json:"core.assets.manufacturer"`
    Barcode         *string    `db:"core.assets.barcode"          json:"core.assets.barcode"`
    PurchaseAmount  *float64   `db:"core.assets.purchase_amount"  json:"core.assets.purchase_amount"`
    PurchaseDate    *time.Time `db:"core.assets.purchase_date"    json:"core.assets.purchase_date"`
    CreatedAt       time.Time  `db:"core.assets.created_at"       json:"core.assets.created_at"`
    UpdatedAt       *time.Time `db:"core.assets.updated_at"       json:"core.assets.updated_at"`
}

// ❌ Incorrect: Not handling nullable fields properly
type Asset struct {
    ID             string    `db:"public.assets.id" json:"id"`
    ModelNumber    string    `db:"public.assets.model_number" json:"model_number"` // Should be *string
    PurchaseAmount float64   `db:"public.assets.purchase_amount" json:"purchase_amount"` // Should be *float64
}
```

### 3. Enum and Status Fields

```go
// ✅ Correct: Use custom types for enums
type AssetStatus string

const (
    AssetStatusActive     AssetStatus = "active"
    AssetStatusInactive   AssetStatus = "inactive"
    AssetStatusMaintenance AssetStatus = "maintenance"
    AssetStatusRetired    AssetStatus = "retired"
    AssetStatusScrapped   AssetStatus = "scrapped"
)

type Asset struct {
    ID     string      `db:"public.assets.id"     json:"id"`
    Name   string      `db:"public.assets.name"   json:"name"`
    Status AssetStatus `db:"public.assets.status" json:"status"`
}

// ❌ Incorrect: Using raw strings for enums
type Asset struct {
    ID     string `db:"public.assets.id" json:"id"`
    Status string `db:"public.assets.status" json:"status"` // Should use custom type
}
```

### 4. Foreign Key Relationships

```go
// ✅ Correct: Proper foreign key mapping
type Asset struct {
    ID             string `db:"public.assets.id"               json:"id"`
    Name           string `db:"public.assets.name"             json:"name"`
    CategoryID     string `db:"public.assets.category_id"      json:"category_id"`
    BranchID       string `db:"public.assets.branch_id"        json:"branch_id"`
    OrganizationID string `db:"public.assets.organization_id"  json:"organization_id"`
    
    // Relationship fields (optional, for joins)
    Category     *Category     `db:"-" json:"category,omitempty"`
    Branch       *Branch       `db:"-" json:"branch,omitempty"`
    Organization *Organization `db:"-" json:"organization,omitempty"`
}

type Category struct {
    ID       string  `db:"public.categories.id"         json:"id"`
    Name     string  `db:"public.categories.name"       json:"name"`
    Icon     string  `db:"public.categories.icon"       json:"icon"`
    ParentID *string `db:"public.categories.parent_id"  json:"parent_id"`
}
```

### 5. Audit Fields Standards

```go
// ✅ Correct: Standard audit fields
type BaseModel struct {
    CreatedAt time.Time  `db:"created_at" json:"created_at"`
    UpdatedAt *time.Time `db:"updated_at" json:"updated_at"`
    CreatedBy string     `db:"created_by" json:"created_by"`
    UpdatedBy *string    `db:"updated_by" json:"updated_by"`
}

type User struct {
    ID       string `db:"public.users.id" json:"id"`
    Username string `db:"public.users.username" json:"username"`
    Email    string `db:"public.users.email" json:"email"`
    BaseModel
}

// ❌ Incorrect: Missing audit fields
type User struct {
    ID       string `db:"public.users.id" json:"id"`
    Username string `db:"public.users.username" json:"username"`
    // Missing audit fields
}
```

### 6. Complex Relationships

```go
// ✅ Correct: Many-to-many relationships
type User struct {
    ID       string `db:"public.users.id"       json:"id"`
    Username string `db:"public.users.username" json:"username"`
    Email    string `db:"public.users.email"    json:"email"`
    BaseModel
}

type Role struct {
    ID   string `db:"public.roles.id"   json:"id"`
    Name string `db:"public.roles.name" json:"name"`
    BaseModel
}

type UserRole struct {
    ID     string `db:"public.user_roles.id"      json:"id"`
    UserID string `db:"public.user_roles.user_id" json:"user_id"`
    RoleID string `db:"public.user_roles.role_id" json:"role_id"`
    BaseModel
}

// ✅ Correct: One-to-many relationships
type Organization struct {
    ID   string `db:"public.organizations.id"   json:"id"`
    Name string `db:"public.organizations.name" json:"name"`
    BaseModel
}

type Branch struct {
    ID             string `db:"public.branches.id"              json:"id"`
    Name           string `db:"public.branches.name"            json:"name"`
    OrganizationID string `db:"public.branches.organization_id" json:"organization_id"`
    BaseModel
    
    // Relationship
    Organization *Organization `db:"-" json:"organization,omitempty"`
}
```

## Database Tag Standards

### 1. Tag Alignment Standards
**CRITICAL**: All struct tags must be properly aligned for readability and consistency.

```go
// ✅ Correct: Properly aligned tags
type User struct {
    ID          string     `db:"public.users.id"            json:"id"`
    Username    string     `db:"public.users.username"      json:"username"`
    Email       string     `db:"public.users.email"         json:"email"`
    Status      string     `db:"public.users.status"        json:"status"`
    CreatedAt   time.Time  `db:"public.users.created_at"    json:"created_at"`
    UpdatedAt   *time.Time `db:"public.users.updated_at"    json:"updated_at"`
    CreatedBy   string     `db:"public.users.created_by"    json:"created_by"`
    UpdatedBy   *string    `db:"public.users.updated_by"    json:"updated_by"`
}

// ❌ Incorrect: Misaligned tags
type User struct {
    ID          string     `db:"public.users.id" json:"id"`
    Username    string     `db:"public.users.username" json:"username"`
    Email       string     `db:"public.users.email" json:"email"`
    Status      string     `db:"public.users.status" json:"status"`
    CreatedAt   time.Time  `db:"public.users.created_at" json:"created_at"`
    UpdatedAt   *time.Time `db:"public.users.updated_at" json:"updated_at"`
    CreatedBy   string     `db:"public.users.created_by" json:"created_by"`
    UpdatedBy   *string    `db:"public.users.updated_by" json:"updated_by"`
}
```

**Alignment Rules:**
1. **Field Names**: Align all field names to the same column
2. **Type Names**: Align all type names to the same column  
3. **DB Tags**: Align all `db:` tags to the same column
4. **JSON Tags**: Align all `json:` tags to the same column
5. **Backticks**: All tags must start and end at the same column positions

### 2. Complete Schema Path
```go
// ✅ Correct: Full schema.table.column path
`db:"public.users.id"`
`db:"public.assets.name"`
`db:"public.categories.parent_id"`

// ❌ Incorrect: Incomplete paths
`db:"id"`
`db:"users.id"`
`db:"name"`
```

### 2. JSON Tag Standards

#### 2.1 Schema-Based JSON Naming (Recommended)
```go
// ✅ Correct: JSON tags follow schema.table.column pattern
type User struct {
    ID          string     `db:"core.users.id"            json:"core.users.id"`
    Username    string     `db:"core.users.username"      json:"core.users.username"`
    Email       string     `db:"core.users.email"         json:"core.users.email"`
    Status      string     `db:"core.users.status"        json:"core.users.status"`
    CreatedAt   time.Time  `db:"core.users.created_at"    json:"core.users.created_at"`
    UpdatedAt   *time.Time `db:"core.users.updated_at"    json:"core.users.updated_at"`
}

type Profile struct {
    ID        string  `db:"core.profile.id"         json:"core.profile.id"`
    UserID    string  `db:"core.profile.user_id"    json:"core.profile.user_id"`
    ImageURL  *string `db:"core.profile.image_url"  json:"core.profile.image_url"`
    Bio       *string `db:"core.profile.bio"        json:"core.profile.bio"`
    Skills    *string `db:"core.profile.skills"     json:"core.profile.skills"`
    CreatedAt time.Time `db:"core.profile.created_at" json:"core.profile.created_at"`
}
```

#### 2.2 Alternative: Simplified JSON Naming (When Schema Context is Clear)
```go
// ✅ Acceptable: Simplified naming when context is clear from API structure
type User struct {
    ID          string     `db:"core.users.id"            json:"id"`
    Username    string     `db:"core.users.username"      json:"username"`
    Email       string     `db:"core.users.email"         json:"email"`
    Status      string     `db:"core.users.status"        json:"status"`
    CreatedAt   time.Time  `db:"core.users.created_at"    json:"created_at"`
    UpdatedAt   *time.Time `db:"core.users.updated_at"    json:"updated_at"`
}

type Profile struct {
    ID        string  `db:"core.profile.id"         json:"id"`
    UserID    string  `db:"core.profile.user_id"    json:"user_id"`
    ImageURL  *string `db:"core.profile.image_url"  json:"image_url"`
    Bio       *string `db:"core.profile.bio"        json:"bio"`
    Skills    *string `db:"core.profile.skills"     json:"skills"`
    CreatedAt time.Time `db:"core.profile.created_at" json:"created_at"`
}
```

#### 2.3 JSON Naming Rules
```go
// ✅ Correct: Consistent JSON naming conventions
`json:"id"`                    // Simple field names
`json:"user_name"`             // Snake case for multi-word fields
`json:"created_at"`            // Timestamp fields
`json:"updated_at"`            // Timestamp fields
`json:"core.users.id"`         // Schema-based naming (preferred)
`json:"core.profile.image_url"` // Schema-based naming (preferred)

// ❌ Incorrect: Inconsistent naming
`json:"ID"`                    // Capital letters
`json:"userName"`              // Camel case
`json:"createdAt"`             // Camel case
`json:"profile_image_url"`     // Mixed naming (should be image_url or core.profile.image_url)
```

#### 2.4 When to Use Schema-Based vs Simplified JSON Naming

**Use Schema-Based JSON Naming (`json:"core.table.column"`) when:**
- Building APIs that serve multiple schemas
- Need to avoid field name conflicts across different tables
- Working with complex data structures that combine multiple tables
- Building generic or reusable API endpoints
- Need clear field origin for debugging and maintenance

**Use Simplified JSON Naming (`json:"column"`) when:**
- API context makes the table/schema clear
- Working with single-table focused endpoints
- Building simple CRUD operations
- Performance is critical (slightly smaller JSON payloads)
- Frontend expects simplified field names

**Example: Mixed Approach**
```go
// For user profile API - simplified naming (context is clear)
type UserProfileResponse struct {
    ID        string  `db:"core.users.id"         json:"id"`
    Username  string  `db:"core.users.username"   json:"username"`
    Email     string  `db:"core.users.email"      json:"email"`
    ImageURL  *string `db:"core.profile.image_url" json:"image_url"`
    Bio       *string `db:"core.profile.bio"      json:"bio"`
}

// For generic data export API - schema-based naming (avoid conflicts)
type DataExportResponse struct {
    UserID    string  `db:"core.users.id"         json:"core.users.id"`
    Username  string  `db:"core.users.username"   json:"core.users.username"`
    ProfileID string  `db:"core.profile.id"       json:"core.profile.id"`
    ImageURL  *string `db:"core.profile.image_url" json:"core.profile.image_url"`
}
```

### 3. Validation Tags
```go
// ✅ Correct: Include validation tags
type User struct {
    ID       string `db:"public.users.id"       json:"id"       validate:"required,uuid"`
    Username string `db:"public.users.username" json:"username" validate:"required,min=3,max=50"`
    Email    string `db:"public.users.email"    json:"email"    validate:"required,email"`
    Status   string `db:"public.users.status"   json:"status"   validate:"required,oneof=active inactive suspended"`
}
```

## File Organization Standards

### 1. Model Files Structure
```
v1/
├── dashboard/
│   ├── assets/
│   │   ├── models/
│   │   │   ├── asset.models.go      # Asset model
│   │   │   ├── category.models.go   # Category model
│   │   │   └── index.go             # Model exports
│   │   └── ...
│   └── ...
├── auth/
│   ├── models/
│   │   ├── user.models.go           # User model
│   │   ├── role.models.go           # Role model
│   │   └── index.go                 # Model exports
│   └── ...
└── shared/
    ├── models/
    │   ├── base.models.go           # Base model with audit fields
    │   ├── common.models.go         # Common types
    │   └── index.go                 # Shared model exports
    └── ...
```

### 2. Model File Naming
```go
// ✅ Correct: Descriptive file names
asset.models.go
category.models.go
user.models.go
base.models.go

// ❌ Incorrect: Generic names
models.go
types.go
structs.go
```

### 3. Model Exports
```go
// ✅ Correct: Clean exports in index.go
package models

// Asset related models
type Asset struct { ... }
type Category struct { ... }
type AssetStatus string

// User related models  
type User struct { ... }
type Role struct { ... }
type UserRole struct { ... }

// Base models
type BaseModel struct { ... }
```

## Best Practices

### 1. Always Use Complete Schema Paths
```go
// ✅ Correct
`db:"public.users.id"`
`db:"public.assets.category_id"`

// ❌ Incorrect
`db:"id"`
`db:"category_id"`
```

### 2. Handle Nullable Fields Properly
```go
// ✅ Correct: Use pointers for nullable fields
UpdatedAt *time.Time `db:"public.users.updated_at" json:"updated_at"`
UpdatedBy *string    `db:"public.users.updated_by" json:"updated_by"`

// ❌ Incorrect: Using non-pointer types for nullable fields
UpdatedAt time.Time `db:"public.users.updated_at" json:"updated_at"`
UpdatedBy string    `db:"public.users.updated_by" json:"updated_by"`
```

### 3. Include Audit Fields
```go
// ✅ Correct: Always include audit fields
type BaseModel struct {
    CreatedAt time.Time  `db:"created_at" json:"created_at"`
    UpdatedAt *time.Time `db:"updated_at" json:"updated_at"`
    CreatedBy string     `db:"created_by" json:"created_by"`
    UpdatedBy *string    `db:"updated_by" json:"updated_by"`
}
```

### 4. Use Custom Types for Enums
```go
// ✅ Correct: Custom types for better type safety
type AssetStatus string
type UserRole string
type OrganizationType string

// ❌ Incorrect: Raw strings
Status string
Role string
Type string
```

### 5. Document Relationships
```go
// ✅ Correct: Document relationships clearly
type Asset struct {
    // Primary fields
    ID           string `db:"public.assets.id"           json:"id"`
    Name         string `db:"public.assets.name"         json:"name"`
    
    // Foreign keys
    CategoryID   string `db:"public.assets.category_id"  json:"category_id"`
    BranchID     string `db:"public.assets.branch_id"     json:"branch_id"`
    
    // Relationships (for joins)
    Category     *Category     `db:"-" json:"category,omitempty"`
    Branch       *Branch       `db:"-" json:"branch,omitempty"`
}
```

## Common Patterns

### 1. Base Model Pattern
```go
type BaseModel struct {
    CreatedAt time.Time  `db:"created_at" json:"created_at"`
    UpdatedAt *time.Time `db:"updated_at" json:"updated_at"`
    CreatedBy string     `db:"created_by" json:"created_by"`
    UpdatedBy *string    `db:"updated_by" json:"updated_by"`
}

// Embed in all models
type User struct {
    ID       string `db:"public.users.id"       json:"id"`
    Username string `db:"public.users.username" json:"username"`
    BaseModel
}
```

### 2. Status Enum Pattern
```go
type AssetStatus string

const (
    AssetStatusActive     AssetStatus = "active"
    AssetStatusInactive   AssetStatus = "inactive"
    AssetStatusMaintenance AssetStatus = "maintenance"
    AssetStatusRetired    AssetStatus = "retired"
    AssetStatusScrapped   AssetStatus = "scrapped"
)

func (s AssetStatus) IsValid() bool {
    switch s {
    case AssetStatusActive, AssetStatusInactive, AssetStatusMaintenance, AssetStatusRetired, AssetStatusScrapped:
        return true
    }
    return false
}
```

### 3. Relationship Pattern
```go
type Asset struct {
    // Primary fields
    ID   string `db:"public.assets.id"           json:"id"`
    Name string `db:"public.assets.name"         json:"name"`
    
    // Foreign keys
    CategoryID string `db:"public.assets.category_id" json:"category_id"`
    
    // Relationship (populated via joins)
    Category *Category `db:"-" json:"category,omitempty"`
}
```

## Validation and Testing

### 1. Model Validation
```go
func (a *Asset) Validate() error {
    if a.Name == "" {
        return errors.New("asset name is required")
    }
    if a.Status != "" && !a.Status.IsValid() {
        return errors.New("invalid asset status")
    }
    return nil
}
```

### 2. Database Tag Validation
```go
func ValidateDBTags(model interface{}) error {
    // Validate that all db tags use complete schema paths
    // Validate that nullable fields use pointers
    // Validate that audit fields are present
    return nil
}
```

## Migration Considerations

### 1. Schema Changes
- Always update model structs when database schema changes
- Maintain backward compatibility where possible
- Document breaking changes clearly

### 2. Type Changes
- Use migration scripts for type changes
- Validate data before applying changes
- Test thoroughly in development environment

### 3. Relationship Changes
- Update foreign key mappings
- Update relationship structs
- Update repository methods

## Summary

This schema mapping standard ensures:
- **Type Safety**: Proper Go types for all database fields
- **Consistency**: Standardized naming and structure
- **Maintainability**: Clear relationships and documentation
- **Scalability**: Patterns that work as the system grows
- **Reliability**: Proper handling of nullable fields and relationships

Follow these standards to maintain a clean, type-safe, and maintainable backend codebase.
