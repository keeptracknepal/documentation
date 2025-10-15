# Items Group Schema Summary

## Overview

The items group schema manages asset inventory and service records using two core tables. This provides complete asset lifecycle management from registration through maintenance history.

## Schema Structure

### assets.items
**Purpose**: Master asset registry containing all physical assets.

**Key Fields**:
- **Identification**: `id`, `name`, `model_number`, `serial_number`, `barcode`
- **Location**: `branch_id`, `location`, `department`
- **Classification**: `category_id`, `manufacturer`, `purchased_from`
- **Purchase**: `purchase_date`, `purchase_amount`, `warranty_time`, `purchase_bill_image`
- **Maintenance**: `last_service_date`, `next_service_date`, `service_interval`
- **Status**: `status`, `is_active`
- **Tracking**: `approved_by`, `created_by`, `updated_by`, timestamps

### assets.items_record
**Purpose**: Service and maintenance history with integrated workflow management.

**Key Fields**:
- **Core**: `id`, `device_id` (→ assets.items), `title`
- **Service**: `service_provider_id`, `service_type`
- **Financial**: `amount`, `approx_next_service_amount`
- **Media**: `bill_image`, `image_url_init`, `image_urls_examination`, `remark`, `examination_report`
- **Dates**: `service_date`, `completion_date`, `approx_next_service_date`
- **Workflow**: `status`, `priority_status`, `current_lifecycle_phase`
- **Users**: `examiner_id`, `updated_by`, `approved_by`, `created_by`, timestamps

## Workflow Phases

### Lifecycle Phases
- `phase_one` - Initial report/request
- `phase_two` - Assignment/acknowledgment  
- `phase_three` - Examination/assessment
- `phase_four` - Decision/approval
- `completed` - Work completed
- `cancelled` - Work cancelled

### Priority Levels
- `low` - Low priority issues
- `medium` - Normal priority issues
- `high` - High priority issues
- `urgent` - Critical priority issues

### Status Values
- `pending` - Awaiting action
- `in_progress` - Work in progress
- `completed` - Work finished
- `cancelled` - Work cancelled
- `on_hold` - Temporarily paused

## Key Relationships

### Primary Relationship
- **assets.items** (1) ←→ (N) **assets.items_record**
- One asset can have multiple service records
- Foreign key: `items_record.device_id` → `items.id`
- Cascade: DELETE CASCADE

### External Dependencies
- **core.branches** → `items.branch_id`
- **assets.categories** → `items.category_id`
- **assets.manufacturers** → `items.manufacturer`
- **assets.vendors** → `items.purchased_from`
- **core.service_providers** → `items_record.service_provider_id`
- **core.users** → All user tracking fields

## Database Files

- `old.database/database/assets/items.sql` - Asset master table
- `old.database/database/assets/items_record.sql` - Service records table

## Core Workflow

1. **Asset Registration**: Assets registered in `assets.items`
2. **Service Request**: Issues logged in `assets.items_record`
3. **Workflow Management**: Records progress through lifecycle phases
4. **Completion**: Service records completed and archived

## Key Features

- **Complete Asset Lifecycle**: From registration to disposal
- **Integrated Workflow**: Phase tracking built into service records
- **Financial Management**: Cost tracking and estimation
- **Media Support**: Image and document management
- **User Accountability**: Complete audit trail
- **Maintenance Scheduling**: Automated service reminders
