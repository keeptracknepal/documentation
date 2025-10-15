# Improved Items Record Schema

## Overview

This document presents an improved version of the `assets.items_record` table with better naming conventions, clearer field purposes, and enhanced functionality while maintaining compatibility with the existing system.

## Current vs Improved Schema

### Table Name
- **Current**: `assets.items_record`
- **Improved**: `assets.maintenance_records`

### Field Improvements

| Current Field | Improved Field | Improvement | Rationale |
|---------------|----------------|-------------|-----------|
| `device_id` | `asset_id` | More accurate terminology | "Asset" is more precise than "device" |
| `title` | `issue_title` | More descriptive | Clearly indicates it's about an issue |
| `service_type` | `maintenance_type` | Consistent terminology | "Maintenance" is more standard |
| `amount` | `estimated_cost` | More descriptive | Indicates it's an estimate |
| `approx_next_service_amount` | `estimated_next_maintenance_cost` | Clearer naming | More descriptive and consistent |
| `bill_image` | `service_receipt_url` | More descriptive | Indicates it's a URL |
| `image_url_init` | `initial_issue_image_url` | Clearer purpose | More descriptive |
| `image_urls_examination` | `examination_image_urls` | Better naming | More descriptive |
| `remark` | `issue_description` | More descriptive | Indicates it's about the issue |
| `service_date` | `maintenance_date` | Consistent terminology | Aligns with maintenance theme |
| `approx_next_service_date` | `estimated_next_maintenance_date` | Consistent naming | More descriptive |
| `status` | `record_status` | More specific | Avoids ambiguity |
| `priority_status` | `priority_level` | More concise | "Level" is clearer than "status" |
| `current_lifecycle_phase` | `current_workflow_phase` | More accurate | "Workflow" is more precise |
| `examiner_id` | `assigned_technician_id` | More descriptive | Indicates assignment |
| `approved_on` | `approved_at` | Consistent naming | Matches other timestamp fields |

## Improved Schema Definition

```sql
-- Improved maintenance records table
CREATE TABLE assets.maintenance_records (
  -- Primary identification
  id uuid NOT NULL DEFAULT gen_random_uuid(),
  asset_id uuid NOT NULL,
  
  -- Issue information
  issue_title character varying(255) NULL,
  issue_description text NULL,
  maintenance_type character varying(100) NULL,
  service_provider_id uuid NULL,
  
  -- Financial information
  estimated_cost numeric(12,2) NULL,
  actual_cost numeric(12,2) NULL, -- New: Track actual vs estimated
  estimated_next_maintenance_cost numeric(12,2) NULL,
  
  -- Media and documentation
  service_receipt_url character varying(500) NULL,
  initial_issue_image_url character varying(500) NULL,
  examination_image_urls text[] NULL,
  examination_report text NULL,
  
  -- Date tracking
  maintenance_date date NULL,
  completion_date date NULL,
  estimated_next_maintenance_date date NULL,
  
  -- Status and workflow
  record_status character varying(50) NULL DEFAULT 'pending',
  priority_level character varying(20) NULL DEFAULT 'normal' CHECK (priority_level IN ('low', 'normal', 'high', 'critical')),
  current_workflow_phase character varying(20) NULL DEFAULT 'reported' CHECK (current_workflow_phase IN ('reported', 'assigned', 'examined', 'decided', 'completed', 'cancelled')),
  
  -- User tracking
  assigned_technician_id uuid NULL,
  updated_by uuid NOT NULL,
  approved_by uuid NULL,
  approved_at timestamp with time zone NULL,
  created_by uuid NOT NULL,
  
  -- Timestamps
  created_at timestamp with time zone NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at timestamp with time zone NULL DEFAULT CURRENT_TIMESTAMP,
  
  -- Constraints
  CONSTRAINT pk_maintenance_records PRIMARY KEY (id),
  CONSTRAINT fk_maintenance_records_asset FOREIGN KEY (asset_id) REFERENCES assets.assets (id) ON DELETE CASCADE,
  CONSTRAINT fk_maintenance_records_service_provider FOREIGN KEY (service_provider_id) REFERENCES core.service_providers (id) ON DELETE SET NULL,
  CONSTRAINT fk_maintenance_records_technician FOREIGN KEY (assigned_technician_id) REFERENCES core.users (id) ON DELETE SET NULL,
  CONSTRAINT fk_maintenance_records_updated_by FOREIGN KEY (updated_by) REFERENCES core.users (id) ON DELETE RESTRICT,
  CONSTRAINT fk_maintenance_records_approved_by FOREIGN KEY (approved_by) REFERENCES core.users (id) ON DELETE SET NULL,
  CONSTRAINT fk_maintenance_records_created_by FOREIGN KEY (created_by) REFERENCES core.users (id) ON DELETE RESTRICT
);
```

## Enhanced Features

### 1. New Fields
- **`actual_cost`**: Track actual vs estimated costs for better financial management
- **`issue_description`**: More detailed issue information (replaces generic `remark`)

### 2. Improved Constraints
- **Check constraints** for `priority_level` and `current_workflow_phase`
- **Standardized naming** for all constraints
- **Better cascade behaviors** for referential integrity

### 3. Enhanced Workflow Phases
- **`reported`** - Issue has been reported
- **`assigned`** - Technician has been assigned
- **`examined`** - Examination has been completed
- **`decided`** - Decision has been made
- **`completed`** - Workflow is complete
- **`cancelled`** - Workflow has been cancelled

### 4. Improved Priority Levels
- **`low`** - Low priority issues
- **`normal`** - Normal priority issues
- **`high`** - High priority issues
- **`critical`** - Critical priority issues

## Index Strategy

```sql
-- Core identification indexes
CREATE INDEX idx_maintenance_records_asset_id ON assets.maintenance_records (asset_id);
CREATE INDEX idx_maintenance_records_service_provider_id ON assets.maintenance_records (service_provider_id);

-- Workflow indexes
CREATE INDEX idx_maintenance_records_workflow_phase ON assets.maintenance_records (current_workflow_phase);
CREATE INDEX idx_maintenance_records_priority_level ON assets.maintenance_records (priority_level);
CREATE INDEX idx_maintenance_records_record_status ON assets.maintenance_records (record_status);

-- User assignment indexes
CREATE INDEX idx_maintenance_records_technician_id ON assets.maintenance_records (assigned_technician_id);
CREATE INDEX idx_maintenance_records_created_by ON assets.maintenance_records (created_by);

-- Composite indexes
CREATE INDEX idx_maintenance_records_asset_phase ON assets.maintenance_records (asset_id, current_workflow_phase);
CREATE INDEX idx_maintenance_records_priority_phase ON assets.maintenance_records (priority_level, current_workflow_phase);
```

## Migration Strategy

### Phase 1: Preparation
1. Create new table with improved naming
2. Set up data migration scripts
3. Create compatibility views for existing applications

### Phase 2: Data Migration
1. Migrate data from old table to new table
2. Map old field names to new field names
3. Verify data integrity

### Phase 3: Application Updates
1. Update application code to use new field names
2. Update all queries and stored procedures
3. Update documentation

### Phase 4: Cleanup
1. Remove old table after verification
2. Remove compatibility views
3. Update all documentation

## Benefits of Improved Design

### 1. Clarity and Readability
- Field names clearly indicate their purpose
- No ambiguity about data content
- Easier for developers to understand and maintain

### 2. Consistency
- Uniform naming patterns
- Predictable naming conventions
- Easier to remember and use

### 3. Enhanced Functionality
- Better cost tracking (estimated vs actual)
- More detailed issue documentation
- Improved workflow phase management
- Enhanced audit trail capabilities

### 4. Industry Standards
- Aligns with common database naming practices
- Easier integration with ORMs and tools
- Better compatibility with reporting tools

