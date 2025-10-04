# KeepTrack Database Service

## Overview

The Database Service is the central data layer for the KeepTrack microservice architecture. It manages PostgreSQL database connections, schema migrations, and provides a unified data access layer for all other services.

## 🏗️ **Service Architecture**

### **Directory Structure**
```
database/
├── connection/           # Database connection management
│   └── supabase.go       # Supabase PostgreSQL connection
├── fetch/                # Data fetching utilities
│   └── fetch.go         # Fetch helper functions
├── functions/            # Database functions and procedures
│   ├── assets.sql       # Asset management functions
│   └── fetched/         # Fetched database objects
│       ├── assets/       # Asset-related functions
│       ├── auth/         # Authentication functions
│       ├── core/         # Core business functions
│       ├── extensions/   # PostgreSQL extensions
│       ├── graphql/      # GraphQL functions
│       ├── realtime/     # Real-time functions
│       ├── storage/      # Storage functions
│       ├── vault/        # Vault functions
│       └── vehicles/     # Vehicle management functions
├── schema/               # Database schemas
│   ├── assets/          # Asset management schema
│   ├── authentication/  # Authentication schema
│   ├── core/            # Core business schema
│   └── vehicles/        # Vehicle management schema
├── update/              # Database migration system
│   ├── index.sql        # Migration index
│   ├── logs.json        # Migration logs
│   └── update.go        # Migration runner
├── main.go              # Database service entry point
├── go.mod               # Go module definition
└── go.sum               # Go module checksums
```

---

## 🗄️ **Database Schemas**

### **1. Core Schema (`core/`)**
**Purpose**: Core business entities and user management

#### **Tables:**
- **`organizations`** - Organization management
- **`branches`** - Branch management within organizations
- **`positions`** - Position/role management
- **`users`** - User account management
- **`access`** - Access control and permissions
- **`profile`** - User profile information

#### **Key Features:**
- **Multi-tenant Architecture**: Organization-based data isolation
- **Role-based Access Control**: Position-based permissions
- **Audit Trails**: Created/updated timestamps and user tracking
- **Data Integrity**: Foreign key relationships and constraints

### **2. Authentication Schema (`authentication/`)**
**Purpose**: JWT-based authentication and security

#### **Tables:**
- **`jwt_request_counters`** - JWT validation attempt tracking

#### **Key Features:**
- **JWT Security**: Request counter for failed validation attempts
- **User Blocking**: Automatic user blocking after failed attempts
- **Security Monitoring**: Track authentication patterns

### **3. Assets Schema (`assets/`)**
**Purpose**: Asset management and tracking

#### **Tables:**
- **`categories`** - Asset category management
- **`items`** - Asset item management
- **`items_history`** - Asset history tracking
- **`items_history_status`** - Asset status management
- **`manufacturer`** - Manufacturer management
- **`model_number`** - Model number management
- **`purchased_from`** - Purchase source management
- **`service_providers`** - Service provider management
- **`shared_bills`** - Shared billing management
- **`tags`** - Asset tagging system

#### **Key Features:**
- **Complete Asset Lifecycle**: From purchase to disposal
- **Service History**: Track maintenance and repairs
- **Status Management**: Asset status tracking
- **Category Hierarchy**: Hierarchical category structure
- **Barcode Support**: Barcode-based asset identification

### **4. Vehicles Schema (`vehicles/`)**
**Purpose**: Vehicle management and tracking

#### **Tables:**
- **`bluebook`** - Vehicle bluebook management
- **`bluebook_renewal`** - Bluebook renewal tracking
- **`insurance_renewal`** - Insurance renewal tracking
- **`pollution_sticker_renewal`** - Pollution sticker tracking

#### **Key Features:**
- **Vehicle Documentation**: Bluebook and registration management
- **Renewal Tracking**: Automatic renewal reminders
- **Compliance Management**: Regulatory compliance tracking

---

## 🔧 **Database Connection Management**

### **Connection Configuration**
```go
// Supabase PostgreSQL Connection
type Database struct {
    DB *sql.DB
}

// Connection Settings
db.SetMaxOpenConns(15)                  // Max open connections
db.SetMaxIdleConns(5)                 // Max idle connections
db.SetConnMaxLifetime(30 * time.Minute) // Connection lifetime
db.SetConnMaxIdleTime(15 * time.Minute) // Idle connection timeout
```

### **Connection Features**
- **Connection Pooling**: Optimized for Supabase pooler
- **Retry Logic**: Exponential backoff for connection failures
- **Health Monitoring**: Connection health checks
- **Timeout Management**: Configurable timeouts

### **Environment Configuration**
```bash
# Database URL (Supabase)
DATABASE_URL="postgresql://user:password@host:port/database?sslmode=require"

# Connection Pool Settings
MAX_OPEN_CONNS=15
MAX_IDLE_CONNS=5
CONN_MAX_LIFETIME=30m
CONN_MAX_IDLE_TIME=15m
```

---

## 📊 **Database Functions**

### **Core Functions**
- **User Management**: Create, read, update, delete users
- **Organization Management**: Organization CRUD operations
- **Branch Management**: Branch operations within organizations
- **Position Management**: Role and position management
- **Access Control**: Permission and access management

### **Asset Functions**
- **Category Management**: Hierarchical category operations
- **Item Management**: Asset item CRUD operations
- **History Tracking**: Asset history and status tracking
- **Service Management**: Maintenance and service tracking
- **Reporting**: Asset statistics and reporting

### **Vehicle Functions**
- **Bluebook Management**: Vehicle documentation
- **Renewal Tracking**: Automatic renewal management
- **Compliance Monitoring**: Regulatory compliance
- **Insurance Management**: Insurance tracking

---

## 🔄 **Migration System**

### **Migration Index (`update/index.sql`)**
```sql
-- Core Tables
\i database/core/users.sql
\i database/core/organizations.sql
\i database/core/branches.sql
\i database/core/positions.sql
\i database/core/access.sql
\i database/core/profile.sql

-- Asset Tables
\i database/assets/categories.sql
\i database/assets/items.sql
\i database/assets/items_history.sql
\i database/assets/items_history_status.sql
\i database/assets/manufacturer.sql
\i database/assets/model_number.sql
\i database/assets/purchased_from.sql
\i database/assets/service_providers.sql
\i database/assets/shared_bills.sql
\i database/assets/tags.sql

-- Vehicle Tables
\i database/vehicles/bluebook.sql
\i database/vehicles/bluebook_renewal.sql
\i database/vehicles/insurance_renewal.sql
\i database/vehicles/pollution_sticker_renewal.sql

-- Authentication Tables
\i database/authentication/jwt_request_counters.sql

-- Extensions
\i database/core/extensions.sql
```

### **Migration Features**
- **Automated Execution**: Automatic migration execution
- **Order Management**: Sequential migration execution
- **Error Handling**: Migration failure handling
- **Logging**: Detailed migration logs
- **Rollback Support**: Migration rollback capabilities

---

## 🛡️ **Security Features**

### **Authentication Security**
- **JWT Request Counting**: Track failed JWT validation attempts
- **User Blocking**: Automatic blocking after failed attempts
- **Security Monitoring**: Authentication pattern analysis
- **Audit Trails**: Security event logging

### **Data Security**
- **Schema Isolation**: Separate schemas for different domains
- **Row Level Security**: Organization-based data isolation
- **Encryption**: Sensitive data encryption
- **Access Control**: Role-based access permissions

### **Connection Security**
- **SSL/TLS**: Encrypted database connections
- **Connection Pooling**: Secure connection management
- **Timeout Protection**: Connection timeout handling
- **Retry Logic**: Secure connection retry mechanisms

---

## 📈 **Performance Optimization**

### **Indexing Strategy**
```sql
-- Core Tables Indexes
CREATE INDEX idx_organizations_name ON core.organizations (name);
CREATE INDEX idx_organizations_is_active ON core.organizations (is_active);
CREATE INDEX idx_branches_organization_id ON core.branches (organization_id);
CREATE INDEX idx_users_email ON core.users (email);

-- Asset Tables Indexes
CREATE INDEX idx_items_branch_id ON assets.items (branch_id);
CREATE INDEX idx_items_category_id ON assets.items (category_id);
CREATE INDEX idx_items_status ON assets.items (status);
CREATE INDEX idx_items_created_at ON assets.items (created_at);

-- Authentication Indexes
CREATE INDEX idx_jwt_request_counters_user_id ON authentication.jwt_request_counters (user_id);
CREATE INDEX idx_jwt_request_counters_is_blocked ON authentication.jwt_request_counters (is_blocked);
```

### **Query Optimization**
- **Foreign Key Indexes**: Optimized relationship queries
- **Composite Indexes**: Multi-column query optimization
- **Partial Indexes**: Conditional index optimization
- **Covering Indexes**: Index-only query optimization

### **Connection Optimization**
- **Connection Pooling**: Efficient connection management
- **Connection Reuse**: Connection lifecycle optimization
- **Timeout Management**: Optimal timeout settings
- **Health Monitoring**: Connection health optimization

---

## 🔄 **Data Relationships**

### **Core Relationships**
```
Organizations (1) ──→ (N) Branches
Organizations (1) ──→ (N) Users
Branches (1) ──→ (N) Positions
Users (1) ──→ (N) Access Records
```

### **Asset Relationships**
```
Organizations (1) ──→ (N) Items
Branches (1) ──→ (N) Items
Categories (1) ──→ (N) Items
Items (1) ──→ (N) Items History
Items (1) ──→ (N) Items History Status
```

### **Vehicle Relationships**
```
Organizations (1) ──→ (N) Bluebooks
Bluebooks (1) ──→ (N) Bluebook Renewals
Bluebooks (1) ──→ (N) Insurance Renewals
```

---

## 🚀 **Deployment & Operations**

### **Production Configuration**
```bash
# Supabase Production URL
DATABASE_URL="postgresql://postgres.nhpxrwymvdwdovbexgvk:HeroBudathoki@aws-1-eu-west-2.pooler.supabase.com:6543/postgres?sslmode=require"

# Connection Pool Settings
MAX_OPEN_CONNS=15
MAX_IDLE_CONNS=5
CONN_MAX_LIFETIME=30m
CONN_MAX_IDLE_TIME=15m
```

### **Migration Deployment**
```bash
# Run database migrations
go run main.go

# Check migration status
cat update/logs.json

# Verify database schema
psql $DATABASE_URL -c "\dt core.*"
psql $DATABASE_URL -c "\dt assets.*"
psql $DATABASE_URL -c "\dt authentication.*"
```

### **Monitoring & Maintenance**
- **Connection Monitoring**: Track connection pool usage
- **Query Performance**: Monitor slow queries
- **Index Maintenance**: Regular index optimization
- **Backup Management**: Automated backup procedures

---

## 📋 **Service Integration**

### **Authentication Service Integration**
- **JWT Validation**: Shared JWT_SECRET for token validation
- **User Management**: User creation and authentication
- **Permission System**: Role-based access control

### **Foundation Service Integration**
- **Dashboard Data**: Core business data for dashboard
- **Asset Management**: Asset CRUD operations
- **User Management**: User profile and settings

### **Media Service Integration**
- **File Metadata**: File information storage
- **Asset Attachments**: Asset file attachments
- **Media Management**: Media file organization

### **Mobile/Web Integration**
- **API Data**: RESTful API data access
- **Real-time Updates**: Database change notifications
- **Offline Support**: Data synchronization

---

## 🎯 **Key Benefits**

### **Centralized Data Management**
- **Single Source of Truth**: Unified data layer
- **Data Consistency**: Consistent data across services
- **Transaction Management**: ACID compliance
- **Backup & Recovery**: Centralized backup management

### **Scalability & Performance**
- **Connection Pooling**: Efficient connection management
- **Query Optimization**: Optimized database queries
- **Indexing Strategy**: Performance-optimized indexes
- **Caching Support**: Database-level caching

### **Security & Compliance**
- **Data Encryption**: Encrypted data storage
- **Access Control**: Role-based permissions
- **Audit Trails**: Complete audit logging
- **Compliance**: Regulatory compliance support

### **Development & Maintenance**
- **Schema Management**: Organized schema structure
- **Migration System**: Automated database updates
- **Documentation**: Comprehensive database documentation
- **Monitoring**: Database performance monitoring

This database service provides a robust, scalable, and secure foundation for the KeepTrack microservice architecture, ensuring data integrity, performance, and security across all services.
