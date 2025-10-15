# Items Record Schema Documentation

## Overview

The `assets.items_record` table is the central hub for maintenance workflow management, combining asset history tracking with notification system workflow. It serves as the primary table for tracking maintenance issues, service records, and workflow progression through a 4-phase notification system.

## Table Relationships

### Parent Table: `assets.items`
- **Relationship**: One-to-Many (items → items_record)
- **Foreign Key**: `device_id` → `assets.items(id)`
- **Cascade**: DELETE CASCADE (when an item is deleted, all its records are deleted)
- **Purpose**: Each item can have multiple maintenance records over time

### Child Table: `assets.items_record_phases`
- **Relationship**: One-to-Many (items_record → items_record_phases)
- **Foreign Key**: `items_record_id` → `assets.items_record(id)`
- **Cascade**: DELETE CASCADE (when a record is deleted, all its phases are deleted)
- **Purpose**: Tracks the complete audit trail of phase transitions for each record

### Related Tables
- `core.service_providers` → `service_provider_id`
- `core.users` → `updated_by`, `approved_by`, `created_by`, `examiner_id`

## Current Table Structure

### Core Identification
- `id` (uuid, PRIMARY KEY) - Unique identifier
- `device_id` (uuid, NOT NULL) - Reference to assets.items table

### Service Information
- `title` (varchar(255)) - Issue or service description
- `service_provider_id` (uuid) - Reference to core.service_providers
- `service_type` (varchar(100)) - Type of service performed

### Financial Data
- `amount` (numeric(12,2)) - Service cost
- `approx_next_service_amount` (numeric(12,2)) - Estimated next service cost

### Media and Documentation
- `bill_image` (varchar(500)) - Bill/receipt image URL
- `image_url_init` (varchar(500)) - Initial issue image URL
- `image_urls_examination` (text[]) - Array of examination photos

### Text Fields
- `remark` (text) - General remarks
- `examination_report` (text) - Detailed examination findings

### Date Tracking
- `service_date` (date) - When service was performed
- `completion_date` (date) - When service was completed
- `approx_next_service_date` (date) - Estimated next service date

### Status and Workflow
- `status` (varchar(50), DEFAULT 'pending') - Current status
- `priority_status` (varchar(20), DEFAULT 'medium') - Priority level
- `current_lifecycle_phase` (varchar(20), DEFAULT 'phase_one') - Current workflow phase

### User Tracking
- `updated_by` (uuid, NOT NULL) - User who last updated
- `approved_by` (uuid) - User who approved
- `approved_on` (timestamp) - When approved
- `created_by` (uuid, NOT NULL) - User who created
- `examiner_id` (uuid) - Assigned technician

### Timestamps
- `created_at` (timestamp, DEFAULT CURRENT_TIMESTAMP)
- `updated_at` (timestamp, DEFAULT CURRENT_TIMESTAMP)

## Improved Schema Design

### Recommended Table Name
- **Current**: `assets.items_record`
- **Recommended**: `assets.maintenance_records`
- **Rationale**: More descriptive, follows plural convention, clearer purpose

### Recommended Column Improvements

#### Core Identification
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `device_id` | `asset_id` | More accurate - these are assets, not just devices |
| `title` | `issue_title` | More descriptive, indicates it's about an issue |

#### Service Information
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `service_type` | `maintenance_type` | More accurate terminology |
| `service_provider_id` | `service_provider_id` | ✓ Keep as is |

#### Financial Data
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `amount` | `estimated_cost` | More descriptive, indicates it's an estimate |
| `approx_next_service_amount` | `estimated_next_maintenance_cost` | More descriptive, consistent terminology |

#### Media and Documentation
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `bill_image` | `service_receipt_url` | More descriptive, indicates it's a URL |
| `image_url_init` | `initial_issue_image_url` | More descriptive and clear |
| `image_urls_examination` | `examination_image_urls` | More descriptive and clear |

#### Text Fields
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `remark` | `issue_description` | More descriptive, indicates it's about the issue |
| `examination_report` | `examination_report` | ✓ Keep as is |

#### Date Tracking
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `service_date` | `maintenance_date` | Consistent terminology |
| `completion_date` | `completion_date` | ✓ Keep as is |
| `approx_next_service_date` | `estimated_next_maintenance_date` | More descriptive, consistent terminology |

#### Status and Workflow
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `status` | `record_status` | More specific, avoids ambiguity |
| `priority_status` | `priority_level` | More concise, "level" is clearer than "status" |
| `current_lifecycle_phase` | `current_workflow_phase` | More accurate terminology |

#### User Tracking
| Current | Recommended | Rationale |
|---------|-------------|-----------|
| `examiner_id` | `assigned_technician_id` | More descriptive, indicates assignment |
| `updated_by` | `updated_by` | ✓ Keep as is |
| `approved_by` | `approved_by` | ✓ Keep as is |
| `created_by` | `created_by` | ✓ Keep as is |
| - | `examined_by` | New: Track who performed the examination |
| - | `examined_at` | New: Track when examination was completed |

### Enhanced Fields (New Additions)

#### Additional Financial Tracking
- `actual_cost` (numeric(12,2)) - Track actual vs estimated costs
- `cost_variance` (numeric(12,2)) - Calculated field for cost analysis

#### Enhanced Documentation
- `issue_description` (text) - More detailed issue information
- `resolution_summary` (text) - Summary of how issue was resolved

#### Workflow Enhancement
- `estimated_resolution_date` (date) - When issue is expected to be resolved
- `actual_resolution_date` (date) - When issue was actually resolved

## Workflow Phase Improvements

### Current Phases
- `phase_one` - Issue reported
- `phase_two` - Technician assigned
- `phase_three` - Examination completed
- `phase_four` - Decision made
- `completed` - Workflow finished
- `cancelled` - Workflow cancelled

### Recommended Phases
- `reported` - Issue reported
- `assigned` - Technician assigned
- `examined` - Examination completed
- `decided` - Decision made
- `completed` - Workflow finished
- `cancelled` - Workflow cancelled

**Rationale**: More descriptive names that clearly indicate the current state rather than generic phase numbers.

## Priority Level Improvements

### Current Priority Levels
- `low` - Low priority
- `medium` - Medium priority
- `high` - High priority
- `urgent` - Urgent priority

### Recommended Priority Levels
- `low` - Low priority
- `normal` - Normal priority (instead of medium)
- `high` - High priority
- `critical` - Critical priority (instead of urgent)

**Rationale**: "Normal" is more intuitive than "medium", and "critical" is more standard than "urgent".

## Foreign Key Relationships

### Current Relationships
- `device_id` → `assets.items(id)` ON DELETE CASCADE
- `service_provider_id` → `core.service_providers(id)` ON DELETE SET NULL
- `updated_by` → `core.users(id)` ON DELETE RESTRICT
- `approved_by` → `core.users(id)` ON DELETE SET NULL
- `created_by` → `core.users(id)` ON DELETE RESTRICT
- `examiner_id` → `core.users(id)` ON DELETE SET NULL

### Improved Relationships
- `asset_id` → `assets.items(id)` ON DELETE CASCADE
- `service_provider_id` → `core.service_providers(id)` ON DELETE SET NULL
- `assigned_technician_id` → `core.users(id)` ON DELETE SET NULL
- `updated_by` → `core.users(id)` ON DELETE RESTRICT
- `approved_by` → `core.users(id)` ON DELETE SET NULL
- `created_by` → `core.users(id)` ON DELETE RESTRICT
- `examined_by` → `core.users(id)` ON DELETE SET NULL

## Index Strategy

### Current Indexes
- `idx_items_record_device_id`
- `idx_items_record_service_provider_id`
- `idx_items_record_service_type`
- `idx_items_record_title`
- `idx_items_record_amount`
- `idx_items_record_approx_next_service_amount`
- `idx_items_record_service_date`
- `idx_items_record_completion_date`
- `idx_items_record_approx_next_service_date`
- `idx_items_record_status`
- `idx_items_record_priority_status`
- `idx_items_record_lifecycle_phase`
- `idx_items_record_updated_by`
- `idx_items_record_approved_by`
- `idx_items_record_created_by`
- `idx_items_record_examiner_id`
- `idx_items_record_created_at`
- `idx_items_record_updated_at`
- `idx_items_record_approved_on`
- `idx_items_record_device_status`
- `idx_items_record_device_lifecycle`
- `idx_items_record_priority_lifecycle`
- `idx_items_record_service_provider_status`
- `idx_items_record_examiner_lifecycle`

### Improved Indexes
- `idx_maintenance_records_asset_id`
- `idx_maintenance_records_service_provider_id`
- `idx_maintenance_records_maintenance_type`
- `idx_maintenance_records_issue_title`
- `idx_maintenance_records_estimated_cost`
- `idx_maintenance_records_actual_cost`
- `idx_maintenance_records_maintenance_date`
- `idx_maintenance_records_completion_date`
- `idx_maintenance_records_estimated_next_maintenance_date`
- `idx_maintenance_records_record_status`
- `idx_maintenance_records_priority_level`
- `idx_maintenance_records_workflow_phase`
- `idx_maintenance_records_updated_by`
- `idx_maintenance_records_approved_by`
- `idx_maintenance_records_created_by`
- `idx_maintenance_records_technician_id`
- `idx_maintenance_records_examined_by`
- `idx_maintenance_records_created_at`
- `idx_maintenance_records_updated_at`
- `idx_maintenance_records_approved_at`
- `idx_maintenance_records_examined_at`
- `idx_maintenance_records_asset_status`
- `idx_maintenance_records_asset_phase`
- `idx_maintenance_records_priority_phase`
- `idx_maintenance_records_service_provider_status`
- `idx_maintenance_records_technician_phase`

## Data Flow and Workflow

### 1. Record Creation
When a maintenance issue is reported:
1. New record created in `assets.items_record` with `current_lifecycle_phase = 'phase_one'`
2. Initial phase record created in `assets.items_record_phases`
3. Notification sent to maintenance department lead

### 2. Phase Progression
Each phase transition:
1. Updates `current_lifecycle_phase` in `assets.items_record`
2. Creates new record in `assets.items_record_phases` for audit trail
3. Triggers appropriate notifications based on phase and cost

### 3. Data Integrity
- **Referential Integrity**: All foreign keys maintain data consistency
- **Cascade Deletes**: Deleting an item removes all related records and phases
- **Audit Trail**: Complete history maintained in phases table

## Common Query Patterns

### Get Active Maintenance Records
```sql
SELECT ir.*, i.name as device_name, i.branch_id
FROM assets.items_record ir
JOIN assets.items i ON ir.device_id = i.id
WHERE ir.current_lifecycle_phase IN ('phase_one', 'phase_two', 'phase_three', 'phase_four')
ORDER BY ir.priority_status, ir.created_at DESC;
```

### Get Complete Workflow History
```sql
SELECT ir.*, irp.lifecycle_phase, irp.status, irp.created_at as phase_created_at
FROM assets.items_record ir
LEFT JOIN assets.items_record_phases irp ON ir.id = irp.items_record_id
WHERE ir.device_id = $1
ORDER BY irp.created_at ASC;
```

### Get Records by Priority and Phase
```sql
SELECT ir.*, i.name as device_name
FROM assets.items_record ir
JOIN assets.items i ON ir.device_id = i.id
WHERE ir.priority_status = 'urgent' 
AND ir.current_lifecycle_phase = 'phase_one';
```

## Usage Notes

- **Central Hub**: This table serves as the main interface for maintenance workflow management
- **Notification Integration**: Workflow fields enable automated status tracking and notifications
- **Audit Trail**: Complete history maintained through phases table relationship
- **Performance**: Comprehensive indexing supports efficient querying by various criteria
- **Data Integrity**: Foreign key constraints ensure referential integrity across the system