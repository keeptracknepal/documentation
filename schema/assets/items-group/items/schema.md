# Items Schema Documentation

## Overview

The `assets.items` table is the master catalog of all physical assets and devices in the system. It serves as the foundation for asset management, maintenance tracking, and service history. Every maintenance record and phase transition ultimately traces back to an item in this table.

## Table Relationships

### Child Tables
- **`assets.items_record`** (One-to-Many)
  - **Foreign Key**: `device_id` → `assets.items(id)`
  - **Cascade**: DELETE CASCADE (when an item is deleted, all its maintenance records are deleted)
  - **Purpose**: Each item can have multiple maintenance records over time

- **`assets.items_record_phases`** (One-to-Many-to-Many)
  - **Connection**: Through `assets.items_record.device_id`
  - **Purpose**: Links all phase transitions back to the original asset

### Related Tables
- `core.branches` → `branch_id` (item location)
- `assets.categories` → `category_id` (item classification)
- `assets.manufacturer` → `manufacturer` (item manufacturer)
- `assets.purchased_from` → `purchased_from` (purchase source)
- `core.users` → `approved_by`, `created_by`, `updated_by`

## Table Structure

### Core Identification
- `id` (uuid, PRIMARY KEY) - Unique identifier
- `name` (varchar(255), NOT NULL) - Asset name/description
- `model_number` (varchar(100)) - Manufacturer model number
- `serial_number` (varchar(100)) - Unique serial number
- `barcode` (varchar(100)) - Barcode for scanning

### Location and Organization
- `branch_id` (uuid, NOT NULL) - Reference to core.branches
- `location` (varchar(255)) - Specific location within branch
- `department` (varchar(100)) - Department responsible for asset

### Classification
- `category_id` (uuid) - Reference to assets.categories
- `manufacturer` (uuid) - Reference to assets.manufacturer
- `purchased_from` (uuid) - Reference to assets.purchased_from

### Purchase Information
- `purchase_date` (date) - When asset was purchased
- `purchase_amount` (numeric(12,2)) - Purchase cost
- `warranty_time` (integer) - Warranty period in months
- `purchase_bill_image` (varchar(500)) - Purchase receipt image

### Service Management
- `last_service_date` (date) - Most recent service date
- `next_service_date` (date) - Scheduled next service
- `service_interval` (integer) - Service interval in days

### Status and Control
- `status` (varchar(50), DEFAULT 'active') - Current asset status
- `is_active` (boolean, DEFAULT true) - Active/inactive flag

### User Tracking
- `approved_by` (uuid) - User who approved the asset
- `approved_on` (timestamp) - When asset was approved
- `created_by` (uuid, NOT NULL) - User who created the record
- `updated_by` (uuid) - User who last updated

### Timestamps
- `created_at` (timestamp, DEFAULT CURRENT_TIMESTAMP)
- `updated_at` (timestamp, DEFAULT CURRENT_TIMESTAMP)

## Foreign Key Relationships

- `branch_id` → `core.branches(id)` ON DELETE CASCADE
- `category_id` → `assets.categories(id)` ON DELETE SET NULL
- `manufacturer` → `assets.manufacturer(id)` ON DELETE SET NULL
- `purchased_from` → `assets.purchased_from(id)` ON DELETE SET NULL
- `approved_by` → `core.users(id)` ON DELETE SET NULL
- `created_by` → `core.users(id)` ON DELETE RESTRICT
- `updated_by` → `core.users(id)` ON DELETE SET NULL

## Indexes

### Single Column Indexes
- `idx_items_branch_id` - branch_id
- `idx_items_category_id` - category_id
- `idx_items_serial_number` - serial_number
- `idx_items_model_number` - model_number
- `idx_items_manufacturer` - manufacturer
- `idx_items_purchased_from` - purchased_from
- `idx_items_purchase_date` - purchase_date
- `idx_items_next_service_date` - next_service_date
- `idx_items_last_service_date` - last_service_date
- `idx_items_status` - status
- `idx_items_location` - location
- `idx_items_department` - department
- `idx_items_barcode` - barcode
- `idx_items_approved_by` - approved_by
- `idx_items_created_by` - created_by
- `idx_items_updated_by` - updated_by
- `idx_items_is_active` - is_active

### Special Indexes
- `idx_items_name_search` - Full-text search on name field
- `idx_items_category_status` - Composite (category_id, status)
- `idx_items_branch_category` - Composite (branch_id, category_id)

## Data Flow and Relationships

### 1. Asset Lifecycle
1. **Asset Creation**: New item created in `assets.items`
2. **Maintenance Issues**: Issues reported create records in `assets.items_record`
3. **Workflow Progression**: Phase transitions tracked in `assets.items_record_phases`
4. **Service Updates**: `last_service_date` and `next_service_date` updated

### 2. Cascade Relationships
- **Item Deletion**: Deleting an item removes all maintenance records and phases
- **Branch Deletion**: Deleting a branch removes all items in that branch
- **Category Deletion**: Items remain but category reference becomes NULL

### 3. Data Integrity
- **Referential Integrity**: All foreign keys maintain data consistency
- **Service Tracking**: Automatic updates to service dates based on maintenance records
- **Status Management**: Centralized status tracking for all assets

## Common Query Patterns

### Get Items with Active Maintenance Issues
```sql
SELECT i.*, COUNT(ir.id) as active_issues
FROM assets.items i
LEFT JOIN assets.items_record ir ON i.id = ir.device_id 
  AND ir.current_lifecycle_phase IN ('phase_one', 'phase_two', 'phase_three', 'phase_four')
WHERE i.is_active = true
GROUP BY i.id
HAVING COUNT(ir.id) > 0
ORDER BY active_issues DESC;
```

### Get Items Due for Service
```sql
SELECT i.*, i.next_service_date - CURRENT_DATE as days_until_service
FROM assets.items i
WHERE i.is_active = true 
AND i.next_service_date <= CURRENT_DATE + INTERVAL '30 days'
ORDER BY i.next_service_date ASC;
```

### Get Complete Asset History
```sql
SELECT 
  i.name as asset_name,
  i.serial_number,
  i.location,
  ir.title as issue_title,
  ir.current_lifecycle_phase,
  ir.priority_status,
  ir.created_at as issue_created
FROM assets.items i
LEFT JOIN assets.items_record ir ON i.id = ir.device_id
WHERE i.id = $1
ORDER BY ir.created_at DESC;
```

### Get Branch Asset Summary
```sql
SELECT 
  b.name as branch_name,
  COUNT(i.id) as total_assets,
  COUNT(CASE WHEN i.status = 'active' THEN 1 END) as active_assets,
  COUNT(ir.id) as active_maintenance_issues
FROM core.branches b
LEFT JOIN assets.items i ON b.id = i.branch_id AND i.is_active = true
LEFT JOIN assets.items_record ir ON i.id = ir.device_id 
  AND ir.current_lifecycle_phase IN ('phase_one', 'phase_two', 'phase_three', 'phase_four')
GROUP BY b.id, b.name
ORDER BY b.name;
```

### Get Asset Performance Metrics
```sql
SELECT 
  i.name,
  i.serial_number,
  COUNT(ir.id) as total_maintenance_issues,
  COUNT(CASE WHEN ir.current_lifecycle_phase = 'completed' THEN 1 END) as completed_issues,
  AVG(EXTRACT(EPOCH FROM (ir.updated_at - ir.created_at))/86400) as avg_resolution_days
FROM assets.items i
LEFT JOIN assets.items_record ir ON i.id = ir.device_id
WHERE i.is_active = true
GROUP BY i.id, i.name, i.serial_number
HAVING COUNT(ir.id) > 0
ORDER BY total_maintenance_issues DESC;
```

## Usage Notes

- **Master Catalog**: This table serves as the authoritative source for all asset information
- **Service Tracking**: Centralized tracking of service schedules and history
- **Location Management**: Branch and location context for all maintenance activities
- **Performance**: Comprehensive indexing supports efficient querying by various criteria
- **Data Integrity**: Foreign key constraints ensure referential integrity across the system
- **Cascade Behavior**: Deleting an item automatically removes all related maintenance data

## Integration with Maintenance System

### With `assets.items_record`
- **Issue Tracking**: Every maintenance issue is linked to a specific item
- **Service History**: Complete maintenance history for each asset
- **Status Correlation**: Item status can be updated based on maintenance issues

### With `assets.items_record_phases`
- **Workflow Context**: All phase transitions are linked back to the original asset
- **Audit Trail**: Complete audit trail of all maintenance activities for each asset
- **Performance Analysis**: Track maintenance patterns and performance by asset
