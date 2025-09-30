# Permission System Summary

## Overview

The KeepTrack permission system is a comprehensive role-based access control (RBAC) system that manages user permissions across different modules and organizational scopes. The system is built around the concept of **positions** that define what users can access and do within the application.

## Core Architecture

### 1. Dual Permission Structure

The permission system uses a dual-layered approach combining:

- **Module-Based Permissions**: Functional area access control
- **Department-Based Permissions**: Organizational structure access control

### 2. Permission Structure

The permission system is organized around these main concepts:

- **Modules**: Different functional areas of the application
- **Departments**: Organizational structure-based access
- **Permissions**: Specific actions users can perform
- **Scopes**: Organizational levels where permissions apply

### 3. JSON Configuration Structure

The permission system uses a structured JSON configuration that defines access control for each position:

```typescript
interface AccessPermission {
    view: boolean;      // Can view/read data
    create: boolean;    // Can create new records
    update: boolean;    // Can modify existing records
    delete: boolean;    // Can delete records
}

interface ModuleAccess {
    scope: 'organization' | 'branches';  // Permission scope level
    permissions: AccessPermission;       // Specific permissions
}

interface DepartmentPositions {
    department_lead: boolean;           // Department leadership position
    department_technician?: boolean;    // Department technician position (optional)
}

interface DepartmentAccess {
    scope: 'organization' | 'branches';  // Department scope level
    positions: DepartmentPositions;      // Department-specific positions
}

interface AccessList {
    modules: {
        employee: ModuleAccess;
        positions: ModuleAccess;
        branches: ModuleAccess;
        dashboard: ModuleAccess;
        assets: ModuleAccess;
        service_provider: ModuleAccess;
        policy: ModuleAccess;
        documents: ModuleAccess;
    };
    departments: {
        maintenance_department: DepartmentAccess;
        management_department: DepartmentAccess;
    };
}
```

## Available Modules

The system supports the following modules:

1. **Assets** - Asset management and tracking
2. **Branches** - Branch/office management
3. **Dashboard** - Main dashboard access
4. **Employee** - Employee management
5. **Positions** - Position/role management
6. **Service Provider** - External service provider management
7. **Policy** - Policy and compliance management
8. **Documents** - Document management and file operations

## Department-Based Access Control

In addition to module-based permissions, the system supports department-based access control that provides organizational structure-based permissions.

### Department Structure

The system supports the following departments:

1. **Maintenance Department** - Handles asset maintenance and technical operations
   - **Department Lead** - Leadership position within maintenance department
   - **Department Technician** - Technical staff responsible for maintenance tasks
2. **Management Department** - Handles administrative and managerial functions
   - **Department Lead** - Leadership position within management department

### Department Access Configuration

```typescript
interface DepartmentPositions {
    department_lead: boolean;           // Department leadership position
    department_technician?: boolean;    // Department technician position (optional)
}

interface DepartmentAccess {
    scope: 'organization' | 'branches';  // Department scope level
    positions: DepartmentPositions;      // Department-specific positions
}

interface PositionAccess {
    modules: {
        // Module-based permissions
        assets: ModuleAccess;
        branches: ModuleAccess;
        dashboard: ModuleAccess;
        employee: ModuleAccess;
        positions: ModuleAccess;
        service_provider: ModuleAccess;
        policy: ModuleAccess;
        documents: ModuleAccess;
    };
    departments: {
        maintenance_department: DepartmentAccess;
        management_department: DepartmentAccess;
    };
}
```

### Department Access Examples

```typescript
const departmentAccessConfig = {
    maintenance_department: {
        scope: 'organization',
        positions: {
            department_lead: true,        // User is a maintenance department lead
            department_technician: false  // User is not a maintenance technician
        }
    },
    management_department: {
        scope: 'organization',
        positions: {
            department_lead: true         // User is a management department lead
        }
    }
};

const fullPositionConfig: PositionAccess = {
    modules: {
        assets: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        branches: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        dashboard: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        employee: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        positions: {
            scope: 'branches',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        documents: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        policy: {
            scope: 'organization',
            permissions: { view: false, create: false, update: false, delete: false }
        },
        service_provider: {
            scope: 'organization',
            permissions: { view: false, create: false, update: false, delete: false }
        }
    },
    departments: {
        maintenance_department: {
            scope: 'organization',
            positions: {
                department_lead: true,
                department_technician: false
            }
        },
        management_department: {
            scope: 'organization',
            positions: {
                department_lead: true
            }
        }
    }
};
```

## Permission System Integration

### How Module and Department Permissions Work Together

The dual permission system provides comprehensive access control by combining:

1. **Module Permissions**: Define what functional areas users can access
2. **Department Permissions**: Define which organizational departments users belong to

### Permission Logic Examples

```typescript
// Example 1: Maintenance Technician
const maintenanceTechnician = {
    modules: {
        assets: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        branches: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        dashboard: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        employee: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        positions: { scope: 'branches', permissions: { view: true, create: true, update: true, delete: true } },
        documents: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        policy: { scope: 'organization', permissions: { view: false, create: false, update: false, delete: false } },
        service_provider: { scope: 'organization', permissions: { view: false, create: false, update: false, delete: false } }
    },
    departments: {
        maintenance_department: {
            scope: 'organization',
            positions: {
                department_lead: true,
                department_technician: false
            }
        },
        management_department: {
            scope: 'organization',
            positions: {
                department_lead: true
            }
        }
    }
};

// Example 2: Management Director
const managementDirector = {
    modules: {
        assets: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        branches: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        dashboard: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        employee: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        positions: { scope: 'branches', permissions: { view: true, create: true, update: true, delete: true } },
        documents: { scope: 'organization', permissions: { view: true, create: true, update: true, delete: true } },
        policy: { scope: 'organization', permissions: { view: false, create: false, update: false, delete: false } },
        service_provider: { scope: 'organization', permissions: { view: false, create: false, update: false, delete: false } }
    },
    departments: {
        maintenance_department: {
            scope: 'organization',
            positions: {
                department_lead: true,
                department_technician: false
            }
        },
        management_department: {
            scope: 'organization',
            positions: {
                department_lead: true
            }
        }
    }
};

// Combined permission checks
const canPerformMaintenance = (user) => {
    return user.departments.maintenance_department.positions.department_lead && 
           user.modules.assets.permissions.update;
};

const canManagePositions = (user) => {
    return user.departments.management_department.positions.department_lead && 
           user.modules.positions.permissions.update;
};
```

## Permission Types

Each module supports four permission types:

- **View** - Read-only access to module data
- **Create** - Ability to add new records
- **Update** - Ability to modify existing records
- **Delete** - Ability to remove records

## Scope Levels

### Organization Scope
- Permissions apply across the entire organization
- Users with organization-level permissions can access data from all branches
- Typically reserved for administrative roles

### Branch Scope
- Permissions are limited to specific branches
- Users can only access data within their assigned branch(es)
- Provides data isolation between different locations

## Default Configuration

The system includes a default permission configuration that serves as a template for new positions:

```typescript
const defaultPositionAccessConfig = {
    modules: {
        assets: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        branches: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        dashboard: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        employee: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        positions: {
            scope: 'branches',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        documents: {
            scope: 'organization',
            permissions: { view: true, create: true, update: true, delete: true }
        },
        policy: {
            scope: 'organization',
            permissions: { view: false, create: false, update: false, delete: false }
        },
        service_provider: {
            scope: 'organization',
            permissions: { view: false, create: false, update: false, delete: false }
        }
    },
    departments: {
        maintenance_department: {
            scope: 'organization',
            positions: {
                department_lead: true,
                department_technician: false
            }
        },
        management_department: {
            scope: 'organization',
            positions: {
                department_lead: true
            }
        }
    }
};
```

## Implementation Details

### Frontend Components

The permission system is implemented through several React components:

#### 1. CreatePositionModal
- Allows creation of new positions with permission configuration
- Provides UI for setting module permissions and scopes
- Includes bulk permission management (Select All/Clear All)
- Validates form data before submission

#### 2. ViewPositionModal
- Displays existing position details and permissions
- Supports inline editing of position information
- Shows permission matrix in a user-friendly format
- Handles permission updates with proper validation

### Key Features

#### Permission Matrix UI
- Grid-based interface showing modules vs permissions
- Visual indicators for granted/denied permissions
- Scope selection (Organization vs Branch) for each module
- Bulk operations for efficient permission management

#### Data Validation
- Ensures all required fields are provided
- Validates permission configurations
- Prevents invalid scope/permission combinations

#### State Management
- Maintains permission state during editing
- Handles form validation and error display
- Manages loading states during API operations

## Usage Examples

### Creating a New Position

```typescript
const positionData = {
    organization_id: "org-123",
    position: "Branch Manager",
    description: "Manages branch operations",
    is_active: true,
    access_list: {
        modules: {
            assets: {
                scope: 'organization',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            branches: {
                scope: 'organization',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            dashboard: {
                scope: 'organization',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            employee: {
                scope: 'organization',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            positions: {
                scope: 'branches',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            documents: {
                scope: 'organization',
                permissions: { view: true, create: true, update: true, delete: true }
            },
            policy: {
                scope: 'organization',
                permissions: { view: false, create: false, update: false, delete: false }
            },
            service_provider: {
                scope: 'organization',
                permissions: { view: false, create: false, update: false, delete: false }
            }
        },
        departments: {
            maintenance_department: {
                scope: 'organization',
                positions: {
                    department_lead: true,
                    department_technician: false
                }
            },
            management_department: {
                scope: 'organization',
                positions: {
                    department_lead: true
                }
            }
        }
    }
};
```

### Checking User Permissions

```typescript
// Check module-based permissions
const canViewAssets = userPosition.access_list.modules.assets.permissions.view;
const hasOrgAccess = userPosition.access_list.modules.branches.scope === 'organization';

// Check department-based permissions
const hasMaintenanceAccess = userPosition.access_list.departments.maintenance_department.positions.department_lead;
const isMaintenanceTechnician = userPosition.access_list.departments.maintenance_department.positions.department_technician;
const hasManagementAccess = userPosition.access_list.departments.management_department.positions.department_lead;

// Combined permission checks
const canManageAssets = canViewAssets && hasMaintenanceAccess;
const canManagePositions = userPosition.access_list.modules.positions.permissions.update && hasManagementAccess;
```

## Security Considerations

1. **Data Isolation**: Branch-scoped permissions ensure users can only access data from their assigned branches
2. **Principle of Least Privilege**: Default configuration starts with minimal permissions
3. **Audit Trail**: All permission changes should be logged for security auditing
4. **Validation**: Server-side validation ensures permission integrity

## Integration Points

The permission system integrates with:

- **Authentication System**: JWT tokens contain user position information
- **API Endpoints**: Backend validates permissions before processing requests
- **UI Components**: Frontend shows/hides features based on user permissions
- **Database**: Permission data is stored and retrieved from the database

## Best Practices

1. **Start with Minimal Permissions**: Begin with the default configuration and add permissions as needed
2. **Regular Audits**: Periodically review and update position permissions
3. **Clear Documentation**: Document what each permission allows users to do
4. **Test Permissions**: Verify that permission restrictions work as expected
5. **Monitor Usage**: Track how permissions are being used in practice

## Future Enhancements

Potential improvements to the permission system:

1. **Dynamic Permissions**: Allow permissions to be modified at runtime
2. **Permission Inheritance**: Support for hierarchical permission structures
3. **Time-based Permissions**: Temporary permissions that expire automatically
4. **Conditional Permissions**: Permissions that depend on specific conditions
5. **Advanced Scoping**: More granular scope options beyond organization/branch
