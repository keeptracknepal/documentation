# @lib Directory Structure Standards

## Overview

The `@lib` directory contains core application logic, API services, utilities, and shared functionality. This document defines the **Layer-Based Structure** for organizing scripts in `@lib`.

## Directory Structure

```
lib/
├── config.ts                          # Application configuration
├── api/
│   ├── v1/                            # API v1 modules
│   │   ├── auth/
│   │   │   ├── dto/
│   │   │   │   ├── index.ts
│   │   │   │   └── auth.dto.ts
│   │   │   ├── models/
│   │   │   │   ├── index.ts
│   │   │   │   └── user.models.ts
│   │   │   ├── service/
│   │   │   │   ├── index.ts
│   │   │   │   └── auth.service.ts
│   │   │   ├── handler/               # Optional
│   │   │   ├── utils/                 # Optional
│   │   │   └── index.ts
│   │   └── users/
│   │       └── [same structure]
│   └── v2/                            # API v2 modules
│       ├── index.ts
│       ├── authentication/
│       │   ├── login/
│       │   │   ├── dto/
│       │   │   ├── models/
│       │   │   ├── service/
│       │   │   ├── handler/
│       │   │   ├── utils/
│       │   │   └── index.ts
│       │   └── guard/
│       │       └── jwt/
│       └── users/
├── hooks/                             # Shared React hooks
├── utils/                             # Shared utilities
└── types/                             # Shared types
```

## Layer Responsibilities

1. **DTO (Data Transfer Object)**: Request/response types for API communication
2. **Models**: Domain models and data structures
3. **Service**: Business logic and API communication
4. **Handler** (Optional): Request handlers that wrap services
5. **Utils** (Optional): Module-specific utility functions

## File Structure Templates

### DTO Layer

**File**: `lib/api/v2/[module]/dto/[module].dto.ts`

```typescript
// ============================================================================
// [MODULE] DTO - Data Transfer Objects
// ============================================================================

export interface [Feature]Request {
  field1: string;
  field2?: number;
}

export interface [Feature]Response {
  success: boolean;
  message: string;
  data?: [Feature]Data;
}

export interface [Feature]Data {
  id: string;
  // ... other fields
}
```

**Index**: `lib/api/v2/[module]/dto/index.ts`

```typescript
export * from './[module].dto';
```

### Models Layer

**File**: `lib/api/v2/[module]/models/[module].models.ts`

```typescript
// ============================================================================
// [MODULE] MODELS - Data Models
// ============================================================================

export interface BaseModel {
  created_at: string;
  updated_at: string;
  created_by?: string;
  updated_by?: string;
}

export interface [Entity] extends BaseModel {
  id: string;
  name: string;
  // ... entity fields
}
```

**Index**: `lib/api/v2/[module]/models/index.ts`

```typescript
export * from './[module].models';
```

### Service Layer

**File**: `lib/api/v2/[module]/service/[module].service.ts`

```typescript
// ============================================================================
// [MODULE] SERVICE - Business Logic
// ============================================================================

import { V2_API_BASE_URL } from '@lib/config';
import type { [Feature]Request, [Feature]Response } from '../dto';
import type { [Entity] } from '../models';

export class [Module]Service {
  private baseUrl: string;

  constructor() {
    this.baseUrl = `${V2_API_BASE_URL}/[module]`;
  }

  async [operation](request: [Feature]Request): Promise<[Feature]Response> {
    try {
      const response = await fetch(`${this.baseUrl}/[endpoint]`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(request),
      });

      const data: [Feature]Response = await response.json();

      if (!response.ok) {
        return {
          success: false,
          message: data.message || '[Operation] failed',
        };
      }

      return data;
    } catch (error) {
      console.error('[Operation] error:', error);
      return {
        success: false,
        message: 'Network error occurred',
      };
    }
  }
}

export const [module]Service = new [Module]Service();
```

**Index**: `lib/api/v2/[module]/service/index.ts`

```typescript
export * from './[module].service';
```

### Handler Layer (Optional)

**File**: `lib/api/v2/[module]/handler/[module].handler.ts`

```typescript
// ============================================================================
// [MODULE] HANDLER - API Handlers
// ============================================================================

import { [module]Service } from '../service';
import type { [Feature]Request, [Feature]Response } from '../dto';

export class [Module]Handler {
  private service = [module]Service;

  async [operation](request: [Feature]Request): Promise<[Feature]Response> {
    try {
      if (!request.field1) {
        return {
          success: false,
          message: '[Field] is required',
        };
      }
      return await this.service.[operation](request);
    } catch (error) {
      return {
        success: false,
        message: error instanceof Error ? error.message : 'Unknown error',
      };
    }
  }
}

export const [module]Handler = new [Module]Handler();
export const [operation] = (request: [Feature]Request) => 
  [module]Handler.[operation](request);
```

### Module Index

**File**: `lib/api/v2/[module]/index.ts`

```typescript
// ============================================================================
// [MODULE] MODULE - V2 API
// ============================================================================

export * from './dto';
export * from './models';
export * from './service';
export * from './handler';
export * from './utils';

/**
 * [Module] Structure:
 * 
 * [module]/
 * ├── index.ts
 * ├── dto/
 * ├── models/
 * ├── service/
 * ├── handler/ (optional)
 * └── utils/ (optional)
 */
```

## Naming Conventions

### Files
- **DTOs**: `[module].dto.ts` (e.g., `login.dto.ts`)
- **Models**: `[module].models.ts` (e.g., `user.models.ts`)
- **Services**: `[module].service.ts` (e.g., `login.service.ts`)
- **Handlers**: `[module].handler.ts` (e.g., `login.handler.ts`)
- **Utils**: `[module].utils.ts` or `[utility].ts` (e.g., `token.ts`)
- **Index**: Always `index.ts`

### Classes
- **Services**: `[Module]Service` (PascalCase, e.g., `LoginService`)
- **Handlers**: `[Module]Handler` (PascalCase, e.g., `LoginHandler`)
- **Singletons**: `[module]Service`, `[module]Handler` (camelCase)

### Interfaces/Types
- **DTOs**: `[Feature]Request`, `[Feature]Response`, `[Feature]Data`
- **Models**: `[Entity]`, `[Entity]With[Relation]`
- **Use PascalCase for all types**

### Functions
- **Service methods**: `[operation]()` (camelCase, e.g., `login()`)
- **Handler methods**: Same as service methods
- **Utils**: `[action][Object]()` (camelCase, e.g., `storeAuthToken()`)

## Import Patterns

### Internal Imports (within module)
```typescript
// In service file
import type { [Feature]Request } from '../dto';
import type { [Entity] } from '../models';

// In handler file
import { [module]Service } from '../service';
```

### External Imports
```typescript
// From config
import { V2_API_BASE_URL } from '@lib/config';

// From other modules
import { getAuthToken } from '@lib/api/v2/authentication/login/utils';
```

### Public API Exports
```typescript
// From module index
import { loginUser, getCurrentUser } from '@lib/api/v2/authentication/login';
```

## Configuration

### API Base URLs
Always use configuration constants from `@lib/config`:

```typescript
// ✅ Good
import { V2_API_BASE_URL } from '@lib/config';
constructor() {
  this.baseUrl = `${V2_API_BASE_URL}/login`;
}

// ❌ Bad
constructor() {
  this.baseUrl = 'http://api.example.com/v2/login';
}
```

### Environment Variables
Access via config file, not directly in services:

```typescript
// ✅ Good - in config.ts
export const V2_API_BASE_URL = 
  process.env.NEXT_PUBLIC_DATABASE_CONNECTION! + '/v2';

// ❌ Bad - in service
const baseUrl = process.env.NEXT_PUBLIC_DATABASE_CONNECTION! + '/v2';
```

## Error Handling

### Service Layer
```typescript
async [operation](request: [Feature]Request): Promise<[Feature]Response> {
  try {
    const response = await fetch(url, options);
    const data: [Feature]Response = await response.json();
    
    if (!response.ok) {
      return {
        success: false,
        message: data.message || 'Operation failed',
      };
    }
    
    return data;
  } catch (error) {
    console.error('[Operation] error:', error);
    return {
      success: false,
      message: 'Network error occurred',
    };
  }
}
```

### Handler Layer
```typescript
async [operation](request: [Feature]Request): Promise<[Feature]Response> {
  try {
    if (!request.requiredField) {
      return {
        success: false,
        message: 'Required field is missing',
      };
    }
    return await this.service.[operation](request);
  } catch (error) {
    return {
      success: false,
      message: error instanceof Error ? error.message : 'Unknown error',
    };
  }
}
```

## Best Practices

### 1. Single Responsibility
- Each layer has one clear purpose
- Services handle business logic only
- Handlers handle request/response validation only

### 2. Type Safety
- Always use TypeScript types
- Export types from DTOs and Models
- Use `type` imports: `import type { ... }`
- Avoid `any` types

### 3. Barrel Exports
- Every directory should have an `index.ts`
- Export all public APIs through index files
- Use `export *` for re-exports

### 4. Documentation
- Document all public methods with JSDoc
- Include module structure comments in index files
- Use clear, descriptive names

### 5. Independence
- v2 modules should be independent from v1
- Use separate token storage keys if needed
- No cross-version dependencies

### 6. Layer Dependencies
```
DTO ← Models (can use DTO types)
  ↑
Service (uses DTO + Models)
  ↑
Handler (uses Service)
  ↑
Utils (independent or uses Models)
```

**Rules:**
- ✅ Service can import from DTO and Models
- ✅ Handler can import from Service, DTO, Models
- ✅ Utils can import from Models (if needed)
- ❌ DTO should not import from other layers
- ❌ Models should not import from other layers
- ❌ Service should not import from Handler

## Migration Guide

When creating new v2 modules:
1. Create directory structure: `lib/api/v2/[module]/`
2. Create layers: `dto/`, `models/`, `service/`, `handler/` (optional), `utils/` (optional)
3. Add `index.ts` in each directory
4. Write DTOs first (define contracts)
5. Write Models (domain entities)
6. Write Service (business logic)
7. Write Handler (if needed for validation/wrapping)
8. Write Utils (if module-specific utilities needed)
9. Export everything through module `index.ts`
10. Document structure in module `index.ts`
