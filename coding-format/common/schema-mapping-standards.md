# Unified Schema Mapping Standards

## Overview
This document defines comprehensive standards for mapping database schemas across all technology stacks in the KeepTrack project: **Golang Backend**, **Next.js Web Application**, and **React Native Mobile Application**. These standards ensure consistency, type safety, and maintainability across the entire codebase.

## Core Principles

### 1. Complete Schema Mapping
- Use complete `schema.table.column` names in database tags
- Include proper data types and constraints
- Document foreign key relationships
- Specify nullable fields with appropriate types
- Include audit fields for tracking

### 2. Type Safety
- Use appropriate types for each technology stack
- Handle nullable fields correctly
- Use custom types for enums and status fields
- Implement proper validation

### 3. Consistency
- Standardize naming conventions across all platforms
- Use consistent tag formats
- Follow established patterns for relationships
- Maintain audit trail standards

### 4. Cross-Platform Alignment
- Ensure API contracts match between backend and frontend
- Maintain consistent field naming across all platforms
- Use shared type definitions where possible
- Document API versioning and breaking changes

## When to Use Schema.Table.Column Naming

### ✅ **USE Schema Naming When:**
- **Database Fields**: All fields that map directly to database columns
- **API Data Transfer**: Fields that represent actual database data
- **Entity Models**: Core business entities (User, Organization, Position, etc.)
- **Data Relationships**: Foreign keys and related entity fields
- **Audit Fields**: Created/Updated timestamps and user tracking

### 🎯 **CRITICAL RULE: Exact Database Mapping**
When using schema naming, you **MUST** use the **exact** database schema name, table name, and column name as they exist in the database. No modifications, abbreviations, or assumptions are allowed.

**Database Structure:**
```sql
-- Example database structure
CREATE SCHEMA core;
CREATE TABLE core.users (
    id VARCHAR(50) PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE SCHEMA assets;
CREATE TABLE assets.items (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    model_number VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Correct Mapping:**
```go
// ✅ Correct: Exact database names used
type User struct {
    ID        string `db:"core.users.id"         json:"core.users.id"`
    Email     string `db:"core.users.email"      json:"core.users.email"`
    FirstName *string `db:"core.users.first_name" json:"core.users.first_name,omitempty"`
    CreatedAt time.Time `db:"core.users.created_at" json:"core.users.created_at"`
}

type Asset struct {
    ID          string  `db:"assets.items.id"           json:"assets.items.id"`
    Name        string  `db:"assets.items.name"         json:"assets.items.name"`
    ModelNumber *string `db:"assets.items.model_number" json:"assets.items.model_number,omitempty"`
    CreatedAt   time.Time `db:"assets.items.created_at" json:"assets.items.created_at"`
}
```

**Incorrect Mapping:**
```go
// ❌ Incorrect: Modified or assumed names
type User struct {
    ID        string `db:"users.id"         json:"users.id"`         // Missing schema
    Email     string `db:"core.user.email"  json:"core.user.email"`  // Wrong table name
    FirstName *string `db:"core.users.fname" json:"core.users.fname"` // Wrong column name
}

// ❌ Incorrect: Abbreviated names
type Asset struct {
    ID    string `db:"assets.item.id"    json:"assets.item.id"`    // Wrong table name
    Name  string `db:"assets.items.nm"   json:"assets.items.nm"`   // Wrong column name
    Model *string `db:"assets.items.mod" json:"assets.items.mod"`  // Wrong column name
}
```

### 🔍 **Validation and Verification**

To ensure strict adherence to exact database mapping, follow these verification steps:

1. **Database Schema Verification:**
   ```sql
   -- Always verify the exact schema, table, and column names
   SELECT table_schema, table_name, column_name, data_type 
   FROM information_schema.columns 
   WHERE table_schema = 'core' AND table_name = 'users';
   ```

2. **Code Review Checklist:**
   - [ ] Schema name matches exactly: `core`, `assets`, `auth`, etc.
   - [ ] Table name matches exactly: `users`, `items`, `organizations`, etc.
   - [ ] Column name matches exactly: `first_name`, `model_number`, `created_at`, etc.
   - [ ] No abbreviations or modifications
   - [ ] No assumptions about naming conventions

3. **Common Mistakes to Avoid:**
   ```go
   // ❌ WRONG: Assumed naming
   FirstName string `db:"core.users.firstname"` // Should be "first_name"
   
   // ❌ WRONG: Abbreviated names
   ID string `db:"core.users.id"` // If actual column is "user_id"
   
   // ❌ WRONG: Missing schema
   Email string `db:"users.email"` // Should be "core.users.email"
   
   // ❌ WRONG: Wrong table name
   Name string `db:"core.user.name"` // Should be "core.users.name"
   ```

4. **Documentation Requirements:**
   - Always document the source database schema
   - Include SQL DDL statements in comments
   - Reference the exact database structure
   - Update documentation when database changes

### ⚠️ **Consequences of Incorrect Mapping**

**Database Query Failures:**
```go
// ❌ This will cause SQL errors
type User struct {
    FirstName string `db:"core.users.firstname"` // Column doesn't exist
}
// SQL Error: column "firstname" does not exist

// ✅ Correct mapping prevents errors
type User struct {
    FirstName string `db:"core.users.first_name"` // Exact column name
}
```

**Data Loss and Corruption:**
- Incorrect column mapping can cause data to be written to wrong columns
- Silent failures where data appears to save but goes to wrong location
- Data integrity issues across the application

**Maintenance Nightmares:**
- Code becomes tightly coupled to incorrect assumptions
- Database changes break application unexpectedly
- Difficult to trace data flow and relationships
- Debugging becomes extremely challenging

**Performance Issues:**
- Incorrect column references can cause full table scans
- Missing indexes due to wrong column names
- Inefficient queries and poor performance

### ❌ **DO NOT USE Schema Naming When:**
- **API Response Wrappers**: `success`, `message`, `error` fields
- **Error Details**: `api.error.code`, `api.error.details` (these are API constructs, not database fields)
- **Pagination Metadata**: `api.pagination.total_count`, `api.pagination.page` (API response structure)
- **UI State Fields**: Component-specific state that doesn't map to database
- **Configuration Objects**: Settings and options that aren't database entities
- **Utility Fields**: Helper fields for processing, not data storage

### 📋 **Examples of Correct Usage:**

#### ✅ **Database Entity Fields (USE Schema Naming):**
```go
// ✅ Correct: Database fields use schema naming
type User struct {
    ID       string `db:"core.users.id"       json:"core.users.id"`
    Email    string `db:"core.users.email"    json:"core.users.email"`
    Username string `db:"core.users.username" json:"core.users.username"`
}
```

#### ❌ **API Response Fields (DO NOT USE Schema Naming):**
```go
// ✅ Correct: API response fields use simple naming
type UserResponse struct {
    Success bool   `json:"success"`    // API wrapper field
    Message string `json:"message"`    // API wrapper field
    Data    *User  `json:"data"`       // Contains schema-named fields
    Error   *ErrorDetail `json:"error"` // API error structure
}

// ✅ Correct: Error details are API constructs, not database fields
type ErrorDetail struct {
    Code    string `json:"code"`    // Simple naming for API fields
    Details string `json:"details"` // Simple naming for API fields
}
```

#### ❌ **Configuration and Utility Fields (DO NOT USE Schema Naming):**
```go
// ✅ Correct: Configuration fields use simple naming
type StorageOptions struct {
    TTL     *int  `json:"ttl"`     // Configuration field
    Encrypt bool  `json:"encrypt"` // Configuration field
    Compress bool `json:"compress"` // Configuration field
}

// ✅ Correct: UI state fields use simple naming
type DashboardState struct {
    Loading bool   `json:"loading"` // UI state field
    Error   string `json:"error"`   // UI state field
    Data    *Data  `json:"data"`    // Contains schema-named fields
}
```

## Technology-Specific Standards

---

## 1. Golang Backend Standards

### 1.1 Basic Struct Mapping

```go
// ✅ Correct: Complete schema mapping with proper types
type User struct {
    ID                  string     `db:"core.users.id"                    json:"core.users.id"`
    Email               string     `db:"core.users.email"                 json:"core.users.email"`
    Phone               string     `db:"core.users.phone"                 json:"core.users.phone"`
    FirstName           *string    `db:"core.users.first_name"            json:"core.users.first_name,omitempty"`
    MiddleName          *string    `db:"core.users.middle_name"           json:"core.users.middle_name,omitempty"`
    LastName            *string    `db:"core.users.last_name"             json:"core.users.last_name,omitempty"`
    Gender              *string    `db:"core.users.gender"                json:"core.users.gender,omitempty"`
    DateOfBirth         *time.Time `db:"core.users.date_of_birth"         json:"core.users.date_of_birth,omitempty"`
    Address             *string    `db:"core.users.address"               json:"core.users.address,omitempty"`
    IsActive            bool       `db:"core.users.is_active"             json:"core.users.is_active"`
    IsEmailVerified     bool       `db:"core.users.is_email_verified"     json:"core.users.is_email_verified"`
    IsPhoneVerified     bool       `db:"core.users.is_phone_verified"     json:"core.users.is_phone_verified"`
    IsLocked            bool       `db:"core.users.is_locked"             json:"core.users.is_locked"`
    LastLoginAt         *time.Time `db:"core.users.last_login_at"         json:"core.users.last_login_at,omitempty"`
    FailedLoginAttempts int        `db:"core.users.failed_login_attempts" json:"core.users.failed_login_attempts"`
    CreatedAt           time.Time  `db:"core.users.created_at"             json:"core.users.created_at"`
    UpdatedAt           time.Time  `db:"core.users.updated_at"             json:"core.users.updated_at"`
    CreatedBy           string     `db:"core.users.created_by"           json:"core.users.created_by"`
    UpdatedBy           *string    `db:"core.users.updated_by"            json:"core.users.updated_by,omitempty"`
}

// ❌ Incorrect: Missing schema, incomplete mapping
type User struct {
    ID       string `db:"id" json:"id"`
    Username string `db:"username" json:"username"`
}
```

### 1.2 Nullable Fields Handling

```go
// ✅ Correct: Use pointers for nullable fields
type Asset struct {
    ID              string     `db:"assets.items.id"               json:"assets.items.id"`
    Name            string     `db:"assets.items.name"             json:"assets.items.name"`
    ModelNumber     *string    `db:"assets.items.model_number"     json:"assets.items.model_number,omitempty"`
    Manufacturer    *string    `db:"assets.items.manufacturer"     json:"assets.items.manufacturer,omitempty"`
    Barcode         *string    `db:"assets.items.barcode"          json:"assets.items.barcode,omitempty"`
    PurchaseAmount  *float64   `db:"assets.items.purchase_amount"  json:"assets.items.purchase_amount,omitempty"`
    PurchaseDate    *time.Time `db:"assets.items.purchase_date"    json:"assets.items.purchase_date,omitempty"`
    CreatedAt       time.Time  `db:"assets.items.created_at"       json:"assets.items.created_at"`
    UpdatedAt       time.Time  `db:"assets.items.updated_at"       json:"assets.items.updated_at"`
}

// ❌ Incorrect: Not handling nullable fields properly
type Asset struct {
    ID             string    `db:"assets.items.id" json:"id"`
    ModelNumber    string    `db:"assets.items.model_number" json:"model_number"` // Should be *string
    PurchaseAmount float64   `db:"assets.items.purchase_amount" json:"purchase_amount"` // Should be *float64
}
```

### 1.3 Enum and Status Fields

```go
// ✅ Correct: Use custom types for enums
type AssetStatus string

const (
    AssetStatusActive      AssetStatus = "active"
    AssetStatusInactive    AssetStatus = "inactive"
    AssetStatusMaintenance AssetStatus = "maintenance"
    AssetStatusRetired     AssetStatus = "retired"
    AssetStatusScrapped    AssetStatus = "scrapped"
)

type Asset struct {
    ID     string      `db:"assets.items.id"     json:"assets.items.id"`
    Name   string      `db:"assets.items.name"   json:"assets.items.name"`
    Status AssetStatus `db:"assets.items.status" json:"assets.items.status"`
}

// ❌ Incorrect: Using raw strings for enums
type Asset struct {
    ID     string `db:"assets.items.id" json:"id"`
    Status string `db:"assets.items.status" json:"status"` // Should use custom type
}
```

### 1.4 Foreign Key Relationships

```go
// ✅ Correct: Proper foreign key mapping
type Asset struct {
    ID             string `db:"assets.items.id"               json:"assets.items.id"`
    Name           string `db:"assets.items.name"             json:"assets.items.name"`
    CategoryID     string `db:"assets.items.category_id"      json:"assets.items.category_id"`
    BranchID       string `db:"assets.items.branch_id"        json:"assets.items.branch_id"`
    OrganizationID string `db:"assets.items.organization_id"  json:"assets.items.organization_id"`
    
    // Relationship fields (optional, for joins)
    Category     *Category     `db:"-" json:"category,omitempty"`
    Branch       *Branch       `db:"-" json:"branch,omitempty"`
    Organization *Organization `db:"-" json:"organization,omitempty"`
}

type Category struct {
    ID       string  `db:"assets.categories.id"         json:"assets.categories.id"`
    Name     string  `db:"assets.categories.name"       json:"assets.categories.name"`
    Icon     string  `db:"assets.categories.icon"       json:"assets.categories.icon"`
    ParentID *string `db:"assets.categories.parent_id"  json:"assets.categories.parent_id,omitempty"`
}
```

### 1.5 Database Tag Alignment Standards

```go
// ✅ Correct: Properly aligned tags
type User struct {
    ID          string     `db:"core.users.id"            json:"core.users.id"`
    Username    string     `db:"core.users.username"      json:"core.users.username"`
    Email       string     `db:"core.users.email"         json:"core.users.email"`
    Status      string     `db:"core.users.status"       json:"core.users.status"`
    CreatedAt   time.Time  `db:"core.users.created_at"   json:"core.users.created_at"`
    UpdatedAt   *time.Time `db:"core.users.updated_at"    json:"core.users.updated_at"`
    CreatedBy   string     `db:"core.users.created_by"    json:"core.users.created_by"`
    UpdatedBy   *string    `db:"core.users.updated_by"    json:"core.users.updated_by"`
}

// ❌ Incorrect: Misaligned tags
type User struct {
    ID          string     `db:"core.users.id" json:"id"`
    Username    string     `db:"core.users.username" json:"username"`
    Email       string     `db:"core.users.email" json:"email"`
}
```

**Alignment Rules:**
1. **Field Names**: Align all field names to the same column
2. **Type Names**: Align all type names to the same column  
3. **DB Tags**: Align all `db:` tags to the same column
4. **JSON Tags**: Align all `json:` tags to the same column
5. **Backticks**: All tags must start and end at the same column positions

---

## 2. Next.js Web Application Standards

### 2.1 TypeScript Interface Mapping

```typescript
// ✅ Correct: Complete interface mapping with schema.table.column naming
export interface User {
  "core.users.id": string;
  "core.users.email": string;
  "core.users.phone": string;
  "core.users.first_name"?: string;
  "core.users.middle_name"?: string;
  "core.users.last_name"?: string;
  "core.users.gender"?: string;
  "core.users.date_of_birth"?: string;
  "core.users.address"?: string;
  "core.users.is_active": boolean;
  "core.users.is_email_verified": boolean;
  "core.users.is_phone_verified": boolean;
  "core.users.is_locked": boolean;
  "core.users.last_login_at"?: string;
  "core.users.failed_login_attempts": number;
  "core.users.created_at": string;
  "core.users.updated_at": string;
  "core.users.created_by": string;
  "core.users.updated_by"?: string;
}

// ❌ Incorrect: Simplified naming without schema context
export interface User {
  id: string;
  email: string;
  firstName?: string; // Should be "core.users.first_name"
  lastName?: string;  // Should be "core.users.last_name"
  // Missing schema.table.column structure
}
```

### 2.2 API Response Types

```typescript
// ✅ Correct: Standardized API response structure with schema naming
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

export interface LoginResponse extends ApiResponse<{
  "auth.tokens.token": string;
  "auth.tokens.user": User;
}> {}

// ❌ Incorrect: Inconsistent response structure
export interface UserResponse {
  user: User;
  status: string; // Should be success: boolean
  msg?: string;   // Should be message?: string
}
```

### 2.3 Form Data Types

```typescript
// ✅ Correct: Form-specific types with schema naming
export interface LoginCredentials {
  "core.users.email_or_mobile": string;
  "core.users.password": string;
}

export interface UserForm {
  "core.users.first_name": string;
  "core.users.middle_name": string;
  "core.users.last_name": string;
  "core.users.gender": string;
  "core.users.date_of_birth": string;
  "core.users.address": string;
  "core.users.phone": string;
}

export interface PersonalInformationProps {
  userForm: UserForm;
  setUserForm: React.Dispatch<React.SetStateAction<UserForm>>;
  isEditing: boolean;
  updating: boolean;
  onEditToggle: () => void;
  onUpdateUser: () => void;
  "core.users.email"?: string;
  "core.users.phone"?: string;
  hasProfileData: boolean;
}

// ❌ Incorrect: Simplified naming without schema context
export interface UserForm {
  firstName: string;    // Should be "core.users.first_name"
  middleName: string;  // Should be "core.users.middle_name"
  lastName: string;    // Should be "core.users.last_name"
  dateOfBirth: string; // Should be "core.users.date_of_birth"
}
```

### 2.4 Component Props Types

```typescript
// ✅ Correct: Component-specific types
export interface FormField {
  name: string;
  label: string;
  type: 'text' | 'email' | 'tel' | 'date' | 'select' | 'textarea';
  required?: boolean;
  disabled?: boolean;
  placeholder?: string;
  options?: { value: string; label: string }[];
  rows?: number;
}

export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

// ❌ Incorrect: Generic or unclear types
export interface FormField {
  name: string;
  type: string; // Should be specific union type
  required: boolean; // Should be optional
}
```

---

## 3. React Native Mobile Application Standards

### 3.1 TypeScript Interface Mapping

```typescript
// ✅ Correct: Mobile-specific interface mapping with schema naming
export interface UserPermissions {
  "core.users.id": string;
  "core.users.email": string;
  "core.users.first_name"?: string;
  "core.users.middle_name"?: string;
  "core.users.last_name"?: string;
  "core.users.phone"?: string;
  "core.users.gender"?: string;
  "core.users.address"?: string;
  "core.users.is_email_verified"?: boolean;
  "core.users.is_phone_verified"?: boolean;
  "core.users.is_locked"?: boolean;
  "core.users.failed_login_attempts"?: number;
  "core.organizations.id": string;
  "core.organizations.name"?: string;
  "core.positions.id"?: string;
  "core.positions.position"?: string;
  "core.profile.skills"?: string;
  "core.access.access_list": {
    [key: string]: {
      permissions: {
        approve: boolean;
        create: boolean;
        delete: boolean;
        update: boolean;
        view: boolean;
      };
      scope: string;
    };
  };
  "core.users.is_active": boolean;
  "core.access.is_access_active"?: boolean;
  "core.users.created_at": string;
  "core.users.updated_at": string;
}

// ❌ Incorrect: Simplified naming without schema context
export interface UserPermissions {
  userId: string;        // Should be "core.users.id"
  userEmail: string;     // Should be "core.users.email"
  firstName?: string;    // Should be "core.users.first_name"
  organizationId: string; // Should be "core.organizations.id"
}
```

### 3.2 API Response Types

```typescript
// ✅ Correct: Mobile API response structure with schema naming
export interface LoginResponse {
  success: boolean;
  data?: {
    "auth.tokens.token": string;
    "auth.tokens.user": {
      "core.users.id": string;
      "core.users.email": string;
      "core.roles.role": string;
      "core.organizations.id": string;
    };
  };
  message?: string;
  error?: string;
}

export interface UserResponse {
  success: boolean;
  data?: {
    "core.users.id": string;
    "core.users.email": string;
    "core.roles.role": string;
    "core.organizations.id": string;
    "core.permissions.permissions": string[];
  };
  message?: string;
  error?: string;
}

// ❌ Incorrect: Simplified naming without schema context
export interface LoginResponse {
  token: string;
  user: User;
  status: 'success' | 'error'; // Should be success: boolean
}
```

### 3.3 Screen and Component Types

```typescript
// ✅ Correct: Screen-specific types with schema naming
export interface DashboardStats {
  "dashboard.stats.total_assets": number;
  "dashboard.stats.pending_items": number;
}

export interface RecentActivity {
  "core.activities.id": string;
  "core.activities.type": 'asset_added' | 'asset_updated' | 'service_completed';
  "core.activities.description": string;
  "core.activities.timestamp": string;
}

export interface DashboardScreenProps {
  navigation: any; // Replace with proper navigation type
  stats: DashboardStats;
  recentActivities: RecentActivity[];
}

// ❌ Incorrect: Simplified naming without schema context
export interface DashboardStats {
  totalAssets: number; // Should be "dashboard.stats.total_assets"
  pendingItems: number; // Should be "dashboard.stats.pending_items"
}
```

### 3.4 Storage and State Types

```typescript
// ✅ Correct: Storage and state management types with schema naming
export interface StorageItem<T = any> {
  "storage.items.key": string;
  "storage.items.value": T;
  "storage.items.timestamp": number;
  "storage.items.expires_at"?: number;
}

export interface StorageOptions {
  "storage.options.expires_in"?: number;
  "storage.options.encrypt"?: boolean;
}

export interface AuthState {
  "auth.state.is_authenticated": boolean;
  "auth.state.user": UserPermissions | null;
  "auth.state.token": string | null;
  "auth.state.loading": boolean;
  "auth.state.error": string | null;
}

// ❌ Incorrect: Simplified naming without schema context
export interface StorageItem {
  key: string; // Should be "storage.items.key"
  value: any; // Should be generic T and "storage.items.value"
  // Missing timestamp and expiration
}
```

---

## 4. Cross-Platform Standards

### 4.1 API Contract Consistency

```typescript
// ✅ Correct: Consistent API contracts with schema naming across platforms
// Backend Go struct
type User struct {
    ID       string `db:"core.users.id"       json:"core.users.id"`
    Email    string `db:"core.users.email"    json:"core.users.email"`
    Username string `db:"core.users.username" json:"core.users.username"`
}

// Next.js Web interface
export interface User {
  "core.users.id": string;
  "core.users.email": string;
  "core.users.username": string;
}

// React Native interface
export interface User {
  "core.users.id": string;
  "core.users.email": string;
  "core.users.username": string;
}

// ❌ Incorrect: Inconsistent field naming
// Backend
type User struct {
    ID       string `db:"core.users.id"       json:"id"`        // Should be "core.users.id"
    Email    string `db:"core.users.email"    json:"email"`     // Should be "core.users.email"
    Username string `db:"core.users.username" json:"username"`  // Should be "core.users.username"
}

// Frontend (inconsistent)
export interface User {
  id: string;        // Should be "core.users.id"
  email: string;      // Should be "core.users.email"
  userName: string;   // Should be "core.users.username"
}
```

### 4.2 Field Naming Conventions

```typescript
// ✅ Correct: Consistent schema.table.column naming across all platforms
// Database: core.users.first_name
// Go: FirstName *string `db:"core.users.first_name" json:"core.users.first_name"`
// TypeScript: "core.users.first_name"?: string;

// ❌ Incorrect: Mixed naming conventions
// Database: core.users.first_name
// Go: FirstName *string `db:"core.users.first_name" json:"firstName"` // Should be "core.users.first_name"
// TypeScript: firstName?: string; // Should be "core.users.first_name"
```

### 4.3 Nullable Field Handling

```typescript
// ✅ Correct: Consistent nullable field handling with schema naming
// Go: *string for nullable fields
type User struct {
    FirstName *string `db:"core.users.first_name" json:"core.users.first_name,omitempty"`
}

// TypeScript: Optional properties for nullable fields
export interface User {
  "core.users.first_name"?: string;
}

// ❌ Incorrect: Inconsistent nullable handling
// Go: string for nullable fields
type User struct {
    FirstName string `db:"core.users.first_name" json:"core.users.first_name"` // Should be *string
}

// TypeScript: Required properties for nullable fields
export interface User {
  "core.users.first_name": string; // Should be optional
}
```

### 4.4 Date and Time Handling

```typescript
// ✅ Correct: Consistent date/time handling with schema naming
// Go: time.Time for timestamps
type User struct {
    CreatedAt time.Time `db:"core.users.created_at" json:"core.users.created_at"`
    UpdatedAt time.Time `db:"core.users.updated_at" json:"core.users.updated_at"`
}

// TypeScript: ISO string format
export interface User {
  "core.users.created_at": string; // ISO 8601 format
  "core.users.updated_at": string; // ISO 8601 format
}

// ❌ Incorrect: Inconsistent date handling
// Go: string for timestamps
type User struct {
    CreatedAt string `db:"core.users.created_at" json:"core.users.created_at"` // Should be time.Time
}

// TypeScript: Date objects
export interface User {
  "core.users.created_at": Date; // Should be string
}
```

---

## 5. File Organization Standards

### 5.1 Golang Backend Structure

```
foundation/v1/
├── authentication/
│   ├── models/
│   │   ├── user.models.go
│   │   ├── role.models.go
│   │   └── index.go
│   └── ...
├── dashboard/
│   ├── assets/
│   │   ├── models/
│   │   │   ├── asset.models.go
│   │   │   ├── category.models.go
│   │   │   └── index.go
│   │   └── ...
│   └── ...
└── shared/
    ├── models/
    │   ├── base.models.go
    │   ├── common.models.go
    │   └── index.go
    └── ...
```

### 5.2 Next.js Web Structure

```
web/app/
├── types/
│   ├── api.types.ts
│   ├── user.types.ts
│   ├── asset.types.ts
│   └── index.ts
├── components/
│   ├── ui/
│   │   ├── Button/
│   │   │   ├── Button.types.ts
│   │   │   └── Button.tsx
│   │   └── ...
│   └── ...
└── ...
```

### 5.3 React Native Mobile Structure

```
mobile/src/
├── types/
│   ├── api.types.ts
│   ├── user.types.ts
│   ├── asset.types.ts
│   └── index.ts
├── lib/
│   ├── authentication/
│   │   ├── types.ts
│   │   └── ...
│   └── ...
└── ...
```

---

## 6. Best Practices

### 6.1 Always Use Complete Schema Paths

```go
// ✅ Correct: Full schema.table.column path
`db:"core.users.id"`
`db:"assets.items.name"`
`db:"core.organizations.name"`

// ❌ Incorrect: Incomplete paths
`db:"id"`
`db:"users.id"`
`db:"name"`
```

### 6.2 Handle Nullable Fields Properly

```go
// ✅ Correct: Use pointers for nullable fields
UpdatedAt *time.Time `db:"core.users.updated_at" json:"updated_at,omitempty"`
UpdatedBy *string    `db:"core.users.updated_by" json:"updated_by,omitempty"`

// ❌ Incorrect: Using non-pointer types for nullable fields
UpdatedAt time.Time `db:"core.users.updated_at" json:"updated_at"`
UpdatedBy string    `db:"core.users.updated_by" json:"updated_by"`
```

### 6.3 Include Audit Fields

```go
// ✅ Correct: Always include audit fields
type BaseModel struct {
    CreatedAt time.Time  `db:"created_at" json:"created_at"`
    UpdatedAt time.Time  `db:"updated_at" json:"updated_at"`
    CreatedBy string     `db:"created_by" json:"created_by"`
    UpdatedBy *string    `db:"updated_by" json:"updated_by,omitempty"`
}
```

### 6.4 Use Custom Types for Enums

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

### 6.5 Document Relationships

```go
// ✅ Correct: Document relationships clearly
type Asset struct {
    // Primary fields
    ID           string `db:"assets.items.id"           json:"assets.items.id"`
    Name         string `db:"assets.items.name"         json:"assets.items.name"`
    
    // Foreign keys
    CategoryID   string `db:"assets.items.category_id"  json:"assets.items.category_id"`
    BranchID     string `db:"assets.items.branch_id"    json:"assets.items.branch_id"`
    
    // Relationships (for joins)
    Category     *Category     `db:"-" json:"category,omitempty"`
    Branch       *Branch       `db:"-" json:"branch,omitempty"`
}
```

---

## 7. Validation and Testing

### 7.1 Model Validation (Golang)

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

### 7.2 TypeScript Validation

```typescript
// ✅ Correct: Type validation functions
export function validateUser(user: Partial<User>): user is User {
  return (
    typeof user.id === 'string' &&
    typeof user.email === 'string' &&
    typeof user.is_active === 'boolean'
  );
}

export function validateLoginCredentials(credentials: any): credentials is LoginCredentials {
  return (
    typeof credentials.email_or_mobile === 'string' &&
    typeof credentials.password === 'string'
  );
}
```

### 7.3 Cross-Platform Testing

```typescript
// ✅ Correct: Shared test data with schema naming
export const mockUser: User = {
  "core.users.id": 'user-123',
  "core.users.email": 'test@example.com',
  "core.users.phone": '+1234567890',
  "core.users.first_name": 'John',
  "core.users.last_name": 'Doe',
  "core.users.is_active": true,
  "core.users.is_email_verified": true,
  "core.users.is_phone_verified": false,
  "core.users.is_locked": false,
  "core.users.failed_login_attempts": 0,
  "core.users.created_at": '2024-01-01T00:00:00Z',
  "core.users.updated_at": '2024-01-01T00:00:00Z',
  "core.users.created_by": 'system',
};
```

---

## 8. Migration Considerations

### 8.1 Schema Changes
- Always update model structs when database schema changes
- Maintain backward compatibility where possible
- Document breaking changes clearly
- Update all platform interfaces simultaneously

### 8.2 Type Changes
- Use migration scripts for type changes
- Validate data before applying changes
- Test thoroughly in development environment
- Update API documentation

### 8.3 Relationship Changes
- Update foreign key mappings
- Update relationship structs
- Update repository methods
- Update frontend interfaces

---

## 9. Summary

This unified schema mapping standard ensures:

- **Type Safety**: Proper types for all database fields across all platforms
- **Consistency**: Standardized naming and structure across Golang, Next.js, and React Native
- **Maintainability**: Clear relationships and documentation
- **Scalability**: Patterns that work as the system grows
- **Reliability**: Proper handling of nullable fields and relationships
- **Cross-Platform Alignment**: Consistent API contracts and data structures

### Key Benefits:
1. **Reduced Bugs**: Type safety prevents runtime errors
2. **Faster Development**: Consistent patterns speed up development
3. **Easier Maintenance**: Clear structure makes changes easier
4. **Better Collaboration**: Standardized patterns improve team efficiency
5. **Future-Proof**: Scalable patterns support growth

Follow these standards to maintain a clean, type-safe, and maintainable codebase across all technology stacks in the KeepTrack project.
