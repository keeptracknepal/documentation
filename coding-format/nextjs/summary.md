# Next.js Web Application Coding Format - Summary

## Overview
This document defines the industry-standard coding format for Next.js web applications. The format emphasizes clean architecture, scalability, maintainability, and follows Next.js 13+ App Router best practices with a well-organized directory structure optimized for enterprise applications.

## Core Principles

### 1. App Router Architecture (Next.js 13+)
- **App Directory**: Modern file-based routing with layouts and pages
- **Server Components**: Default server-side rendering for optimal performance
- **Client Components**: Strategic use of client-side interactivity
- **Streaming**: Progressive loading and Suspense boundaries

### 2. Layered Architecture
```
┌─────────────────┐
│     Pages       │ ← App Router Pages (UI Routes)
├─────────────────┤
│    Layouts      │ ← Shared Layouts and Templates
├─────────────────┤
│   Components    │ ← Reusable UI Components
├─────────────────┤
│   Dashboard/    │ ← Dashboard-specific logic
│   Console/      │ ← Console-specific logic
├─────────────────┤
│     Auth/       │ ← Authentication (shared)
├─────────────────┤
│    Common/      │ ← Shared utilities and API
├─────────────────┤
│     Utils       │ ← Utility Functions
├─────────────────┤
│     Types       │ ← TypeScript Definitions
└─────────────────┘
```

### 3. Separation of Concerns
- **Single Responsibility**: Each component has one reason to change
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Interface Segregation**: Small, focused interfaces
- **Composition over Inheritance**: Use composition for component relationships

## Directory Structure

```
web/
├── app/                              # Next.js App Router
│   ├── (main)/                       # Route groups
│   │   ├── layout.tsx               # Main layout
│   │   ├── page.tsx                  # Home page
│   │   ├── login/
│   │   │   ├── layout.tsx           # Login layout
│   │   │   └── page.tsx             # Login page
│   │   └── register/
│   │       └── page.tsx             # Register page
│   ├── dashboard/                    # Dashboard section
│   │   ├── layout.tsx               # Dashboard layout
│   │   ├── page.tsx                 # Dashboard home
│   │   ├── assets/                  # Assets module
│   │   │   ├── page.tsx             # Assets list page
│   │   │   ├── create/              # Asset creation
│   │   │   │   ├── page.tsx        # Create asset page
│   │   │   │   └── components/     # Page-specific components
│   │   │   │       ├── AssetForm.tsx
│   │   │   │       └── index.ts
│   │   │   ├── [id]/               # Dynamic asset routes
│   │   │   │   ├── page.tsx        # Asset detail page
│   │   │   │   ├── edit/
│   │   │   │   │   └── page.tsx    # Edit asset page
│   │   │   │   └── components/     # Asset-specific components
│   │   │   └── components/         # Shared asset components
│   │   │       ├── AssetCard.tsx
│   │   │       ├── AssetList.tsx
│   │   │       └── index.ts
│   │   ├── employees/              # Employees module
│   │   │   ├── page.tsx
│   │   │   └── components/
│   │   └── settings/              # Settings module
│   │       ├── page.tsx
│   │       └── components/
│   ├── console/                    # Console section
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── organizations/          # Console modules
│   ├── globals.css                 # Global styles
│   └── layout.tsx                  # Root layout
├── components/                     # Global reusable components
│   ├── ui/                         # Base UI components
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.types.ts
│   │   │   ├── Button.styles.ts
│   │   │   └── index.ts
│   │   ├── Input/
│   │   │   ├── Input.tsx
│   │   │   ├── Input.types.ts
│   │   │   ├── Input.styles.ts
│   │   │   └── index.ts
│   │   ├── Modal/
│   │   │   ├── Modal.tsx
│   │   │   ├── Modal.types.ts
│   │   │   ├── Modal.styles.ts
│   │   │   └── index.ts
│   │   └── index.ts                # UI components barrel export
│   ├── forms/                      # Form components
│   │   ├── FormField/
│   │   │   ├── FormField.tsx
│   │   │   ├── FormField.types.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── layout/                     # Layout components
│   │   ├── Header/
│   │   │   ├── Header.tsx
│   │   │   ├── Header.types.ts
│   │   │   └── index.ts
│   │   ├── Sidebar/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Sidebar.types.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   └── index.ts                    # Global components barrel export
├── lib/                            # Core application logic
│   ├── auth/                       # Authentication (shared)
│   │   ├── AuthProvider.tsx
│   │   ├── useAuth.ts
│   │   ├── login.ts
│   │   ├── register.ts
│   │   └── index.ts
│   ├── dashboard/                  # Dashboard-specific logic
│   │   ├── services/               # Dashboard services
│   │   │   ├── assets/
│   │   │   │   ├── assets.service.ts
│   │   │   │   ├── assets.types.ts
│   │   │   │   └── index.ts
│   │   │   ├── employees/
│   │   │   │   ├── employees.service.ts
│   │   │   │   ├── employees.types.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   ├── hooks/                  # Dashboard-specific hooks
│   │   │   ├── useAssets.ts
│   │   │   ├── useEmployees.ts
│   │   │   └── index.ts
│   │   ├── utils/                  # Dashboard-specific utilities
│   │   │   ├── dateConverter.ts
│   │   │   ├── numberFormatter.ts
│   │   │   └── index.ts
│   │   ├── types/                  # Dashboard-specific types
│   │   │   ├── assets.ts
│   │   │   ├── employees.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── console/                    # Console-specific logic
│   │   ├── services/               # Console services
│   │   │   ├── organizations/
│   │   │   │   ├── organizations.service.ts
│   │   │   │   ├── organizations.types.ts
│   │   │   │   └── index.ts
│   │   │   ├── users/
│   │   │   │   ├── users.service.ts
│   │   │   │   ├── users.types.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   ├── hooks/                  # Console-specific hooks
│   │   │   ├── useOrganizations.ts
│   │   │   ├── useUsers.ts
│   │   │   └── index.ts
│   │   ├── utils/                  # Console-specific utilities
│   │   │   ├── accessControl.ts
│   │   │   ├── permissions.ts
│   │   │   └── index.ts
│   │   ├── types/                  # Console-specific types
│   │   │   ├── organizations.ts
│   │   │   ├── users.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── common/                     # Shared utilities
│   │   ├── client/                 # HTTP client configuration
│   │   │   ├── client.ts
│   │   │   ├── endpoints.ts
│   │   │   └── types.ts
│   │   ├── hooks/                  # Shared hooks
│   │   │   ├── useHttp.ts
│   │   │   ├── useLocalStorage.ts
│   │   │   └── index.ts
│   │   ├── utils/                  # Shared utilities
│   │   │   ├── imageUpload.ts
│   │   │   ├── validation.ts
│   │   │   └── index.ts
│   │   ├── types/                  # Shared types
│   │   │   ├── http.ts
│   │   │   ├── common.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   └── config.ts                   # Application configuration
├── public/                         # Static assets
│   ├── icons/
│   ├── images/
│   └── logos/
├── styles/                         # Global styles
│   └── globals.css
├── types/                          # TypeScript declarations
├── next.config.js                  # Next.js configuration
├── tailwind.config.js              # Tailwind CSS configuration
├── tsconfig.json                   # TypeScript configuration
└── package.json                    # Dependencies
```

## Component Standards

### 1. Component Structure
Each component should follow this structure:

```typescript
// ComponentName.tsx
'use client'; // Only when needed for client components

import React from 'react';
import { ComponentNameProps } from './ComponentName.types';
import { componentStyles } from './ComponentName.styles';

export const ComponentName: React.FC<ComponentNameProps> = ({
  // Props destructuring
}) => {
  // Hooks
  // State
  // Effects
  // Handlers
  // Render
  return (
    <div className={componentStyles.container}>
      {/* Component content */}
    </div>
  );
};

export default ComponentName;
```

### 2. Component Types
```typescript
// ComponentName.types.ts
export interface ComponentNameProps {
  // Required props
  title: string;
  // Optional props
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  // Event handlers
  onClick?: () => void;
  // Children
  children?: React.ReactNode;
}

export interface ComponentNameState {
  // Internal state types
  isLoading: boolean;
  error: string | null;
}
```

### 3. Component Styles
```typescript
// ComponentName.styles.ts
import { cn } from '@/lib/utils';

export const componentStyles = {
  container: 'base-styles',
  header: 'header-styles',
  content: 'content-styles',
  footer: 'footer-styles',
};

export const getVariantStyles = (variant: string) => {
  const variants = {
    primary: 'primary-styles',
    secondary: 'secondary-styles',
  };
  return variants[variant] || variants.primary;
};
```

### 4. Component Index
```typescript
// index.ts
export { ComponentName } from './ComponentName';
export type { ComponentNameProps } from './ComponentName.types';
export { componentStyles } from './ComponentName.styles';
```

## Service Layer Standards

### 1. API Service Structure
```typescript
// services/api/client.ts
import axios from 'axios';
import { API_BASE_URL } from '@/lib/config';

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use((config) => {
  // Add auth token
  return config;
});

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Handle errors
    return Promise.reject(error);
  }
);
```

### 2. Domain Service Structure
```typescript
// services/assets/assets.service.ts
import { apiClient } from '../api/client';
import { Asset, CreateAssetRequest, UpdateAssetRequest } from './assets.types';

export class AssetsService {
  private baseUrl = '/assets';

  async getAssets(params?: GetAssetsParams): Promise<Asset[]> {
    const response = await apiClient.get(`${this.baseUrl}`, { params });
    return response.data;
  }

  async getAssetById(id: string): Promise<Asset> {
    const response = await apiClient.get(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async createAsset(data: CreateAssetRequest): Promise<Asset> {
    const response = await apiClient.post(this.baseUrl, data);
    return response.data;
  }

  async updateAsset(id: string, data: UpdateAssetRequest): Promise<Asset> {
    const response = await apiClient.put(`${this.baseUrl}/${id}`, data);
    return response.data;
  }

  async deleteAsset(id: string): Promise<void> {
    await apiClient.delete(`${this.baseUrl}/${id}`);
  }
}

export const assetsService = new AssetsService();
```

## Custom Hooks Standards

### 1. Data Fetching Hooks
```typescript
// hooks/useAssets.ts
import { useState, useEffect } from 'react';
import { assetsService } from '@/lib/services/assets';
import { Asset } from '@/lib/types';

export const useAssets = () => {
  const [assets, setAssets] = useState<Asset[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchAssets = async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await assetsService.getAssets();
      setAssets(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchAssets();
  }, []);

  return {
    assets,
    loading,
    error,
    refetch: fetchAssets,
  };
};
```

### 2. Form Hooks
```typescript
// hooks/useForm.ts
import { useState, useCallback } from 'react';

export const useForm = <T extends Record<string, any>>(
  initialValues: T,
  validationSchema?: any
) => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});

  const setValue = useCallback((field: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [field]: value }));
  }, []);

  const setFieldTouched = useCallback((field: keyof T) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  return {
    values,
    errors,
    touched,
    setValue,
    setFieldTouched,
    reset,
  };
};
```

## Theme and Styling Standards

### 1. Design System Structure
```typescript
// lib/theme/colors.ts
export const colors = {
  primary: {
    50: '#eff6ff',
    500: '#3b82f6',
    900: '#1e3a8a',
  },
  gray: {
    50: '#f9fafb',
    100: '#f3f4f6',
    500: '#6b7280',
    900: '#111827',
  },
  semantic: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6',
  },
};
```

### 2. Component Styles
```typescript
// lib/theme/components.ts
export const components = {
  button: {
    base: 'inline-flex items-center justify-center font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2',
    variants: {
      primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
      secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
    },
    sizes: {
      sm: 'px-3 py-1.5 text-sm',
      md: 'px-4 py-2 text-base',
      lg: 'px-6 py-3 text-lg',
    },
  },
};
```

## Utility Functions Standards

### 1. Utility Structure
```typescript
// lib/utils/dateConverter.ts
export const dateConverter = {
  toNepali: (date: Date): string => {
    // Implementation
  },
  toEnglish: (date: string): Date => {
    // Implementation
  },
  format: (date: Date, format: string): string => {
    // Implementation
  },
};
```

### 2. Number Formatting
```typescript
// lib/utils/numberFormatter.ts
export const numberFormatter = {
  currency: (amount: number, currency = 'NPR'): string => {
    return new Intl.NumberFormat('en-NP', {
      style: 'currency',
      currency,
    }).format(amount);
  },
  number: (num: number): string => {
    return new Intl.NumberFormat('en-NP').format(num);
  },
};
```

## TypeScript Standards

### 1. Type Definitions
```typescript
// lib/types/api.ts
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  error?: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    total: number;
    limit: number;
    offset: number;
    hasMore: boolean;
  };
}
```

### 2. Component Props
```typescript
// lib/types/common.ts
export interface BaseComponentProps {
  className?: string;
  children?: React.ReactNode;
}

export interface LoadingState {
  isLoading: boolean;
  error: string | null;
}
```

## Best Practices

### 1. Performance Optimization
- Use Server Components by default
- Implement proper loading states
- Use React.memo for expensive components
- Implement proper error boundaries
- Use Suspense for data fetching

### 2. Code Organization
- Keep components small and focused
- Use barrel exports for clean imports
- Implement proper TypeScript types
- Follow consistent naming conventions
- Use meaningful file and folder names

### 3. Accessibility
- Use semantic HTML elements
- Implement proper ARIA attributes
- Ensure keyboard navigation
- Provide proper focus management
- Use proper color contrast

### 4. Testing
- Write unit tests for utilities
- Test component behavior
- Mock API calls properly
- Test error scenarios
- Implement integration tests

## Migration Strategy

### Phase 1: Foundation
1. Create new directory structure
2. Move existing components to new structure
3. Update import paths
4. Implement new component standards

### Phase 2: Services
1. Refactor API calls to service layer
2. Implement proper error handling
3. Add TypeScript types
4. Create custom hooks

### Phase 3: Optimization
1. Implement Server Components
2. Add proper loading states
3. Optimize bundle size
4. Implement caching strategies

This coding format ensures scalability, maintainability, and follows Next.js best practices for enterprise applications.
