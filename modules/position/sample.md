# Position Access Control - Sample JSON Configurations

This document provides sample JSON configurations for different position types based on the dual permission system (module-based and department-based access control).

## Sample Position Configurations

### 1. Maintenance Technician

A technical staff member responsible for asset maintenance tasks.

```json
{
  "organization_id": "org-123",
  "position": "Maintenance Technician",
  "description": "Technical staff responsible for maintenance tasks",
  "is_active": true,
  "access_list": {
    "modules": {
      "assets": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": false
        }
      },
      "dashboard": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "branches": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "employee": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "positions": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "service_provider": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "policy": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "budget": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      }
    },
    "departments": {
      "maintenance_department": true,
      "maintenance_technician": true,
      "management_department": false
    }
  }
}
```

### 2. Branch Manager

A management position responsible for branch operations and oversight.

```json
{
  "organization_id": "org-123",
  "position": "Branch Manager",
  "description": "Manages branch operations and oversees staff",
  "is_active": true,
  "access_list": {
    "modules": {
      "assets": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "branches": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": true,
          "delete": false
        }
      },
      "budget": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": false
        }
      },
      "dashboard": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "employee": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": false
        }
      },
      "positions": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "service_provider": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "policy": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      }
    },
    "departments": {
      "maintenance_department": false,
      "maintenance_technician": false,
      "management_department": true
    }
  }
}
```

### 3. Organization Administrator

A high-level administrative position with organization-wide access.

```json
{
  "organization_id": "org-123",
  "position": "Organization Administrator",
  "description": "Full administrative access across the organization",
  "is_active": true,
  "access_list": {
    "modules": {
      "assets": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "branches": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "budget": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "dashboard": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "employee": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "positions": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "service_provider": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "policy": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      }
    },
    "departments": {
      "maintenance_department": true,
      "maintenance_technician": false,
      "management_department": true
    }
  }
}
```

### 4. Read-Only Analyst

A position with view-only access for reporting and analysis purposes.

```json
{
  "organization_id": "org-123",
  "position": "Business Analyst",
  "description": "Read-only access for reporting and analysis",
  "is_active": true,
  "access_list": {
    "modules": {
      "assets": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "branches": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "budget": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "dashboard": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "employee": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "positions": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "service_provider": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "policy": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      }
    },
    "departments": {
      "maintenance_department": false,
      "maintenance_technician": false,
      "management_department": false
    }
  }
}
```

### 5. Service Provider Coordinator

A position focused on managing external service providers.

```json
{
  "organization_id": "org-123",
  "position": "Service Provider Coordinator",
  "description": "Manages relationships with external service providers",
  "is_active": true,
  "access_list": {
    "modules": {
      "assets": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "branches": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "budget": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "dashboard": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "employee": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "positions": {
        "scope": "organization",
        "permissions": {
          "view": false,
          "create": false,
          "update": false,
          "delete": false
        }
      },
      "service_provider": {
        "scope": "branches",
        "permissions": {
          "view": true,
          "create": true,
          "update": true,
          "delete": true
        }
      },
      "policy": {
        "scope": "organization",
        "permissions": {
          "view": true,
          "create": false,
          "update": false,
          "delete": false
        }
      }
    },
    "departments": {
      "maintenance_department": true,
      "maintenance_technician": false,
      "management_department": false
    }
  }
}
```

## Permission Logic Examples

### Checking Combined Permissions

```javascript
// Example: Check if user can perform maintenance tasks
function canPerformMaintenance(userPosition) {
  return userPosition.access_list.departments.maintenance_department && 
         userPosition.access_list.modules.assets.permissions.update;
}

// Example: Check if user can approve budgets
function canApproveBudgets(userPosition) {
  return userPosition.access_list.departments.management_department && 
         userPosition.access_list.modules.budget.permissions.update;
}

// Example: Check if user has organization-wide access
function hasOrganizationAccess(userPosition) {
  return userPosition.access_list.modules.branches.scope === 'organization';
}

// Example: Check if user is a maintenance technician
function isMaintenanceTechnician(userPosition) {
  return userPosition.access_list.departments.maintenance_technician;
}
```

## Default Configuration Template

```json
{
  "modules": {
    "assets": {
      "scope": "branches",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "branches": {
      "scope": "organization",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "budget": {
      "scope": "organization",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "dashboard": {
      "scope": "branches",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "employee": {
      "scope": "organization",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "positions": {
      "scope": "organization",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "service_provider": {
      "scope": "branches",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    },
    "policy": {
      "scope": "organization",
      "permissions": {
        "view": false,
        "create": false,
        "update": false,
        "delete": false
      }
    }
  },
  "departments": {
    "maintenance_department": false,
    "maintenance_technician": false,
    "management_department": false
  }
}
```

## Notes

- All permissions start as `false` by default (principle of least privilege)
- Department permissions are boolean flags (true/false)
- Module permissions include scope (organization/branches) and CRUD operations
- Combined permission checks should consider both module and department access
- Organization scope allows access across all branches
- Branch scope limits access to specific branch(es) only
