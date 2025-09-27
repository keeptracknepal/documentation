# Next.js Coding Format Documentation

## Overview
This documentation provides comprehensive coding standards and best practices for Next.js web applications, focusing on code organization, structure, and maintainability.

## Documentation Structure

### 📋 [Summary](./summary.md)
Complete overview of the Next.js coding format including:
- Core principles and architecture
- Directory structure guidelines
- Component standards
- Service layer patterns
- Best practices and migration strategy

### 🧩 [Component Standards](./component-standards.md)
Detailed guidelines for component development:
- Component architecture and categories
- Component structure templates
- Naming conventions and patterns
- Performance optimization
- Testing strategies
- Accessibility guidelines

### 🔧 [Service Layer Standards](./service-layer-standards.md)
Comprehensive service layer implementation:
- HTTP client configuration
- Domain service structure
- Custom hooks for data management
- Error handling patterns
- Caching and performance optimization
- Form handling utilities

## Quick Start Guide

### 1. Project Setup
```bash
# Create new Next.js project
npx create-next-app@latest my-app --typescript --tailwind --eslint

# Install additional dependencies
npm install clsx tailwind-merge
```

### 2. Directory Structure
```
web/
├── app/                    # Next.js App Router
├── components/            # Reusable components
├── lib/                   # Core application logic
│   ├── auth/             # Authentication
│   ├── dashboard/        # Dashboard-specific logic
│   ├── console/          # Console-specific logic
│   ├── common/           # Shared utilities
│   └── config.ts         # Configuration
├── public/               # Static assets
└── styles/              # Global styles
```

### 3. Component Example
```typescript
// app/dashboard/assets/components/AssetCard.tsx
'use client';

import React from 'react';
import { AssetCardProps } from './AssetCard.types';

export const AssetCard: React.FC<AssetCardProps> = ({
  asset,
  onEdit,
  onDelete,
  className,
  ...props
}) => {
  return (
    <div className={className} {...props}>
      <h3>{asset.name}</h3>
      <p>{asset.branch_name}</p>
      <p>{asset.status}</p>
    </div>
  );
};
```

### 4. Service Example
```typescript
// lib/dashboard/services/assets/assets.service.ts
import { httpClient } from '@/lib/common/client/client';
import { Asset, CreateAssetRequest } from './assets.types';

export class AssetsService {
  private baseUrl = '/dashboard/pages/assets';

  async getAssets(): Promise<Asset[]> {
    const response = await httpClient.get<Asset[]>(this.baseUrl);
    return response;
  }

  async createAsset(data: CreateAssetRequest): Promise<Asset> {
    const response = await httpClient.post<Asset>(this.baseUrl, data);
    return response;
  }
}

export const assetsService = new AssetsService();
```

### 5. Custom Hook Example
```typescript
// lib/dashboard/hooks/useAssets.ts
import { useState, useEffect } from 'react';
import { assetsService } from '@/lib/dashboard/services';

export const useAssets = () => {
  const [assets, setAssets] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchAssets = async () => {
      try {
        setLoading(true);
        const data = await assetsService.getAssets();
        setAssets(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchAssets();
  }, []);

  return { assets, loading, error };
};
```

## Key Principles

### 1. **App Router First**
- Use Next.js 13+ App Router for all new projects
- Leverage Server Components by default
- Use Client Components only when necessary

### 2. **Module Independence**
- Dashboard and console work independently
- Shared authentication layer
- Common utilities for shared functionality

### 3. **Component Composition**
- Build small, focused components
- Use composition over inheritance
- Implement proper TypeScript types

### 4. **Service Layer Pattern**
- Abstract HTTP calls into services
- Use custom hooks for data management
- Implement proper error handling

### 5. **Code Organization**
- Clear separation between modules
- Consistent file and folder naming
- Proper import/export patterns

## Migration Strategy

### Phase 1: Foundation
1. Set up new directory structure
2. Create base components and services
3. Implement authentication layer
4. Update existing components

### Phase 2: Service Layer
1. Refactor HTTP calls to services
2. Create custom hooks
3. Implement error handling
4. Add caching strategies

### Phase 3: Optimization
1. Convert to Server Components
2. Implement proper loading states
3. Optimize performance
4. Add testing coverage

## Best Practices

### ✅ Do's
- Use TypeScript for all components
- Implement proper error boundaries
- Follow accessibility guidelines
- Use semantic HTML elements
- Implement proper loading states
- Write comprehensive tests

### ❌ Don'ts
- Don't mix server and client logic
- Don't ignore accessibility
- Don't skip error handling
- Don't use any types unnecessarily
- Don't forget to optimize performance

## Resources

### Documentation
- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [TypeScript Documentation](https://www.typescriptlang.org/docs)

### Tools
- [ESLint](https://eslint.org) - Code linting
- [Prettier](https://prettier.io) - Code formatting
- [Husky](https://typicode.github.io/husky) - Git hooks
- [Jest](https://jestjs.io) - Testing framework
- [Testing Library](https://testing-library.com) - Component testing

This documentation ensures consistent, maintainable, and scalable Next.js application development.