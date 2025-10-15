# Schema Relationships

## Primary Relationships

### assets.items ↔ assets.items_record
- **Type**: One-to-Many
- **Foreign Key**: `items_record.device_id` → `items.id`
- **Cascade**: DELETE CASCADE
- **Purpose**: Each asset can have multiple service records

## External Dependencies

### assets.items Dependencies
| Field | References | Cascade | Purpose |
|-------|------------|---------|---------|
| `branch_id` | core.branches | CASCADE | Asset location |
| `category_id` | assets.categories | SET NULL | Asset classification |
| `manufacturer` | assets.manufacturers | SET NULL | Manufacturer info |
| `purchased_from` | assets.vendors | SET NULL | Purchase source |
| `approved_by` | core.users | SET NULL | Approval tracking |
| `created_by` | core.users | RESTRICT | Creation tracking |
| `updated_by` | core.users | SET NULL | Update tracking |

### assets.items_record Dependencies
| Field | References | Cascade | Purpose |
|-------|------------|---------|---------|
| `device_id` | assets.items | CASCADE | Asset reference |
| `service_provider_id` | core.service_providers | SET NULL | Service provider |
| `examiner_id` | core.users | SET NULL | Assigned technician |
| `updated_by` | core.users | RESTRICT | Update tracking |
| `approved_by` | core.users | SET NULL | Approval tracking |
| `created_by` | core.users | RESTRICT | Creation tracking |

## Data Flow

### Asset Lifecycle
1. Asset created in `assets.items`
2. Service requests logged in `assets.items_record`
3. Records progress through workflow phases
4. Service completed and archived

### User Interactions
- **Asset Management**: Create, update, approve assets
- **Service Assignment**: Assign technicians to records
- **Workflow Control**: Progress records through phases
- **Approval Process**: Approve completed work

## Integration Points

### Notification System
- Status changes trigger notifications
- Phase transitions generate alerts
- User assignments create events

### Reporting
- Asset inventory from `assets.items`
- Service history from `assets.items_record`
- Performance metrics across both tables
- Financial tracking and analysis