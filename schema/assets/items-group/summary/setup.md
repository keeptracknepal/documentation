# Schema Setup Guide

## Database Files

### Core Tables
- `old.database/database/assets/items.sql` - Asset master table
- `old.database/database/assets/items_record.sql` - Service records table

## Table Creation Order

1. **Prerequisites** (must exist first):
   - `core.users`
   - `core.branches`
   - `assets.categories`
   - `assets.manufacturers`
   - `assets.vendors`
   - `core.service_providers`

2. **Core Tables**:
   - `assets.items` (asset master)
   - `assets.items_record` (service records)

## Key Indexes

### assets.items
- Primary key: `id`
- Foreign keys: `branch_id`, `category_id`, `manufacturer`, `purchased_from`
- Status: `status`, `is_active`
- Maintenance: `next_service_date`, `last_service_date`

### assets.items_record
- Primary key: `id`
- Foreign key: `device_id` (→ assets.items)
- Workflow: `status`, `priority_status`, `current_lifecycle_phase`
- Users: `examiner_id`, `created_by`, `updated_by`
- Dates: `service_date`, `completion_date`

## Configuration

### Default Values
- `items.status`: 'active'
- `items.is_active`: true
- `items_record.status`: 'pending'
- `items_record.priority_status`: 'medium'
- `items_record.current_lifecycle_phase`: 'phase_one'

### Constraints
- All UUID fields use `gen_random_uuid()`
- Timestamps default to `CURRENT_TIMESTAMP`
- Foreign keys have appropriate cascade behaviors

## Initial Data

### Required Reference Data
- At least one branch in `core.branches`
- At least one user in `core.users`
- Asset categories in `assets.categories`
- Manufacturers in `assets.manufacturers`
- Vendors in `assets.vendors`
- Service providers in `core.service_providers`

## Maintenance

### Regular Tasks
- Update `next_service_date` based on `service_interval`
- Archive completed service records
- Update asset status based on service history
- Clean up old media files

### Monitoring
- Track service completion rates
- Monitor asset maintenance schedules
- Review user assignment patterns
- Analyze cost trends
