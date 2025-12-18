# Schema Mapping Standards

## Overview
Simple, consistent field naming across all technology stacks: **Golang Backend**, **Next.js Web**, and **React Native Mobile**.

## Core Principle
**Use simple field names with database documentation in comments.**

## Standards

### ✅ **Correct Approach**
```go
// Go Backend
type User struct {
    ID        string     `db:"id"         json:"id"`                    // core.users.id
    Email     string     `db:"email"     json:"email"`                // core.users.email
    FirstName *string    `db:"first_name" json:"first_name,omitempty"` // core.users.first_name
    CreatedAt time.Time  `db:"created_at" json:"created_at"`           // core.users.created_at
}
```

```typescript
// Next.js Web Application
export interface User {
  id: string;                    // core.users.id
  email: string;                 // core.users.email
  firstName?: string;            // core.users.first_name
  createdAt: string;            // core.users.created_at
}

// React Native Mobile Application
export interface User {
  id: string;                    // core.users.id
  email: string;                 // core.users.email
  firstName?: string;            // core.users.first_name
  createdAt: string;            // core.users.created_at
}
```

### ❌ **Incorrect Approach**
```go
// Too complex
type User struct {
    CoreUsersID string `db:"core_users_id" json:"core_users_id"`
}

// Missing documentation
type User struct {
    ID string `db:"id" json:"id"` // Missing: // core.users.id
}
```

## Rules

### 1. **Field Names**
- Use simple, intuitive names: `id`, `email`, `firstName`
- Avoid complex prefixes: `coreUsersId` ❌
- Use camelCase for TypeScript: `firstName`
- Use PascalCase for Go: `FirstName`

### 2. **Database Documentation**
- Always document database mapping in comments
- Format: `// schema.table.column`
- Example: `// core.users.id`

### 3. **Nullable Fields**
- Go: Use pointers `*string`, `*time.Time`
- TypeScript: Use optional `string?`, `Date?`
- Add `omitempty` tag for JSON serialization

### 4. **Audit Fields**
Include in all entities:
- `createdAt` // schema.table.created_at
- `updatedAt` // schema.table.updated_at  
- `createdBy` // schema.table.created_by
- `updatedBy` // schema.table.updated_by

## Examples

### Go Backend
```go
type User struct {
    ID                  string     `db:"id"                    json:"id"`                    // core.users.id
    Email               string     `db:"email"                 json:"email"`                // core.users.email
    Phone               string     `db:"phone"                 json:"phone"`                // core.users.phone
    FirstName            *string    `db:"first_name"            json:"first_name,omitempty"`  // core.users.first_name
    LastName             *string    `db:"last_name"             json:"last_name,omitempty"`   // core.users.last_name
    IsActive             bool       `db:"is_active"             json:"is_active"`             // core.users.is_active
    CreatedAt            time.Time  `db:"created_at"            json:"created_at"`             // core.users.created_at
    UpdatedAt            time.Time  `db:"updated_at"            json:"updated_at"`             // core.users.updated_at
    CreatedBy            string     `db:"created_by"            json:"created_by"`            // core.users.created_by
    UpdatedBy            *string    `db:"updated_by"            json:"updated_by,omitempty"` // core.users.updated_by
}
```

### Next.js Web Application
```typescript
// User Interface
export interface User {
  id: string;                    // core.users.id
  email: string;                 // core.users.email
  phone: string;                 // core.users.phone
  firstName?: string;            // core.users.first_name
  lastName?: string;             // core.users.last_name
  isActive: boolean;             // core.users.is_active
  createdAt: string;            // core.users.created_at
  updatedAt: string;            // core.users.updated_at
  createdBy: string;            // core.users.created_by
  updatedBy?: string;           // core.users.updated_by
}

// Asset Interface
export interface Asset {
  id: string;                    // assets.items.id
  name: string;                  // assets.items.name
  modelNumber?: string;          // assets.items.model_number
  createdAt: string;            // assets.items.created_at
  updatedAt: string;            // assets.items.updated_at
  createdBy: string;            // assets.items.created_by
  updatedBy?: string;           // assets.items.updated_by
}

// API Response Types
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  message?: string;
  error?: {
    code: string;
    details: string;
  };
}

export interface UserResponse extends ApiResponse<User> {}
export interface AssetResponse extends ApiResponse<Asset> {}
```

### React Native Mobile Application
```typescript
// User Interface
export interface User {
  id: string;                    // core.users.id
  email: string;                 // core.users.email
  phone: string;                 // core.users.phone
  firstName?: string;            // core.users.first_name
  lastName?: string;             // core.users.last_name
  isActive: boolean;             // core.users.is_active
  createdAt: string;            // core.users.created_at
  updatedAt: string;            // core.users.updated_at
  createdBy: string;            // core.users.created_by
  updatedBy?: string;           // core.users.updated_by
}

// Asset Interface
export interface Asset {
  id: string;                    // assets.items.id
  name: string;                  // assets.items.name
  modelNumber?: string;          // assets.items.model_number
  createdAt: string;            // assets.items.created_at
  updatedAt: string;            // assets.items.updated_at
  createdBy: string;            // assets.items.created_by
  updatedBy?: string;           // assets.items.updated_by
}

// Screen Props
export interface DashboardScreenProps {
  navigation: any;
  user: User;
  assets: Asset[];
}

// State Management
export interface AuthState {
  isAuthenticated: boolean;
  user: User | null;
  token: string | null;
  loading: boolean;
  error: string | null;
}
```

## Benefits

- ✅ **Simple**: Easy to read and use
- ✅ **Fast**: Quick development
- ✅ **Compatible**: Works with all tools
- ✅ **Traceable**: Database mapping documented
- ✅ **Type Safe**: Full TypeScript/Go support
- ✅ **Performance**: No string parsing needed

## When to Use

- **Database Fields**: All fields mapping to database columns
- **API Data Transfer**: Fields representing database data
- **Entity Models**: Core business entities
- **Data Relationships**: Foreign keys and related fields
- **Audit Fields**: Created/Updated timestamps and user tracking

## Summary

Use simple field names with database documentation in comments. This approach provides the best balance of simplicity, maintainability, and traceability across all technology stacks.
