# Desktop Application - Directory Structure Standards

## Overview

This document outlines a clean, organized directory structure for the KeepTrack desktop application. The desktop is a **frontend-only** application that connects to the Go backend API, with support for multiple API versions and shared themes.

## Core Principles

1. **Clean & Simple**: Easy to understand at a glance
2. **Version-Based**: API versions (v1, v2) clearly organized
3. **Shared Resources**: Themes and common code shared between web and desktop
4. **Frontend-Only**: All business logic via backend API
5. **Modular**: Easy to add/remove features

## Complete Directory Structure

```
desktop/
├── app/                                   # Next.js App Router (Pages)
│   ├── (auth)/                           # Auth routes group
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   │
│   ├── dashboard/                        # Dashboard routes
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── assets/
│   │   │   ├── page.tsx
│   │   │   ├── create/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       ├── page.tsx
│   │   │       └── edit/
│   │   │           └── page.tsx
│   │   ├── employees/
│   │   ├── categories/
│   │   └── settings/
│   │
│   ├── layout.tsx                        # Root layout
│   └── page.tsx                          # Home page
│
├── components/                           # UI Components (Shared with Web)
│   ├── ui/                              # Base UI components
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── card.tsx
│   │   ├── table.tsx
│   │   ├── modal.tsx
│   │   └── index.ts
│   │
│   ├── forms/                           # Form components
│   │   ├── form-field.tsx
│   │   ├── form-group.tsx
│   │   └── index.ts
│   │
│   ├── layout/                          # Layout components
│   │   ├── header.tsx
│   │   ├── sidebar.tsx
│   │   ├── footer.tsx
│   │   └── index.ts
│   │
│   ├── desktop/                         # Desktop-only components
│   │   ├── title-bar.tsx               # Custom window title bar
│   │   ├── native-menu.tsx             # Native menu integration
│   │   ├── system-tray.tsx             # System tray UI
│   │   └── index.ts
│   │
│   └── index.ts
│
├── lib/                                 # Business Logic (Versioned by API)
│   ├── v1/                             # API Version 1
│   │   ├── auth/
│   │   │   ├── auth.service.ts        # Auth API calls
│   │   │   ├── auth.hooks.ts          # Auth hooks (useAuth, useLogin)
│   │   │   ├── auth.types.ts          # Auth types
│   │   │   └── index.ts
│   │   │
│   │   ├── assets/
│   │   │   ├── assets.service.ts      # Assets API calls
│   │   │   ├── assets.hooks.ts        # Assets hooks (useAssets, useAsset)
│   │   │   ├── assets.types.ts        # Asset types
│   │   │   └── index.ts
│   │   │
│   │   ├── employees/
│   │   │   ├── employees.service.ts
│   │   │   ├── employees.hooks.ts
│   │   │   ├── employees.types.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── categories/
│   │   │   ├── categories.service.ts
│   │   │   ├── categories.hooks.ts
│   │   │   ├── categories.types.ts
│   │   │   └── index.ts
│   │   │
│   │   ├── common/
│   │   │   ├── http-client.ts         # HTTP client for v1
│   │   │   ├── interceptors.ts        # Request/response interceptors
│   │   │   ├── endpoints.ts           # API endpoints
│   │   │   └── index.ts
│   │   │
│   │   └── index.ts                   # v1 barrel export
│   │
│   ├── v2/                            # API Version 2 (Future)
│   │   ├── auth/
│   │   ├── assets/
│   │   ├── common/
│   │   └── index.ts
│   │
│   ├── desktop/                       # Desktop-specific logic
│   │   ├── electron-api.ts           # Electron API wrapper
│   │   ├── window.ts                 # Window controls
│   │   ├── dialog.ts                 # Native dialogs
│   │   ├── storage.ts                # Local storage
│   │   ├── notification.ts           # Native notifications
│   │   └── index.ts
│   │
│   ├── utils/                        # Shared utilities
│   │   ├── format.ts                 # Formatting functions
│   │   ├── validation.ts             # Validation functions
│   │   ├── date.ts                   # Date utilities
│   │   ├── number.ts                 # Number utilities
│   │   └── index.ts
│   │
│   └── hooks/                        # Shared hooks
│       ├── use-debounce.ts
│       ├── use-local-storage.ts
│       ├── use-async.ts
│       └── index.ts
│
├── themes/                           # Theme System (Shared with Web)
│   ├── default/
│   │   ├── colors.ts
│   │   ├── typography.ts
│   │   ├── spacing.ts
│   │   └── index.ts
│   ├── dark/
│   │   └── index.ts
│   ├── light/
│   │   └── index.ts
│   ├── theme-provider.tsx
│   └── index.ts
│
├── electron/                         # Electron Main Process
│   ├── main.ts                       # Entry point
│   ├── window.ts                     # Window management
│   ├── menu.ts                       # Application menu
│   ├── tray.ts                       # System tray
│   ├── ipc.ts                        # IPC handlers
│   └── preload.ts                    # Preload script
│
├── types/                            # TypeScript Types
│   ├── global.d.ts
│   ├── electron.d.ts
│   └── index.ts
│
├── styles/                           # Global Styles
│   ├── globals.css
│   └── desktop.css
│
├── public/                           # Static Assets
│   ├── icons/
│   ├── images/
│   └── fonts/
│
├── config/                           # Configuration
│   ├── app.config.ts                # App configuration
│   └── env.ts                       # Environment variables
│
├── .env.local                        # Environment variables
├── next.config.js                    # Next.js config
├── electron-builder.yml              # Electron builder config
├── tsconfig.json                     # TypeScript config
├── package.json
└── README.md
```

## Key Concepts

### 1. Versioned API Structure (lib/v1, lib/v2)

Each API version is self-contained with:
- **Services**: API calls to backend
- **Hooks**: React hooks for data fetching
- **Types**: TypeScript definitions
- **Common**: HTTP client, interceptors, endpoints

```typescript
// lib/v1/assets/assets.service.ts
import { httpClient } from '../common/http-client';
import type { Asset, CreateAssetDTO } from './assets.types';

export const assetsService = {
  getAll: () => httpClient.get<Asset[]>('/api/v1/dashboard/assets'),
  getById: (id: string) => httpClient.get<Asset>(`/api/v1/dashboard/assets/${id}`),
  create: (data: CreateAssetDTO) => httpClient.post<Asset>('/api/v1/dashboard/assets', data),
  update: (id: string, data: CreateAssetDTO) => httpClient.put<Asset>(`/api/v1/dashboard/assets/${id}`, data),
  delete: (id: string) => httpClient.delete(`/api/v1/dashboard/assets/${id}`),
};
```

```typescript
// lib/v1/assets/assets.hooks.ts
import { useState, useEffect } from 'react';
import { assetsService } from './assets.service';
import type { Asset } from './assets.types';

export function useAssets() {
  const [assets, setAssets] = useState<Asset[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const fetchAssets = async () => {
    setLoading(true);
    setError(null);
    try {
      const data = await assetsService.getAll();
      setAssets(data);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchAssets();
  }, []);

  return { assets, loading, error, refetch: fetchAssets };
}

export function useAsset(id: string) {
  const [asset, setAsset] = useState<Asset | null>(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!id) return;
    
    setLoading(true);
    assetsService.getById(id)
      .then(setAsset)
      .finally(() => setLoading(false));
  }, [id]);

  return { asset, loading };
}
```

```typescript
// lib/v1/common/http-client.ts
import axios from 'axios';

const BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8080';

export const httpClient = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Auth interceptor
httpClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('auth_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Error interceptor
httpClient.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### 2. Theme System (Shared)

```typescript
// themes/default/colors.ts
export const colors = {
  primary: {
    50: '#f0f9ff',
    100: '#e0f2fe',
    500: '#0ea5e9',
    600: '#0284c7',
    900: '#0c4a6e',
  },
  gray: {
    50: '#f9fafb',
    100: '#f3f4f6',
    500: '#6b7280',
    900: '#111827',
  },
  success: '#10b981',
  error: '#ef4444',
  warning: '#f59e0b',
};
```

```typescript
// themes/theme-provider.tsx
import { createContext, useContext, useState } from 'react';

type Theme = 'light' | 'dark';

const ThemeContext = createContext<{
  theme: Theme;
  setTheme: (theme: Theme) => void;
}>({ theme: 'light', setTheme: () => {} });

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <div data-theme={theme}>
        {children}
      </div>
    </ThemeContext.Provider>
  );
}

export const useTheme = () => useContext(ThemeContext);
```

### 3. Desktop-Specific Features

```typescript
// lib/desktop/electron-api.ts
export const electronAPI = {
  window: {
    minimize: () => window.electronAPI?.window.minimize(),
    maximize: () => window.electronAPI?.window.maximize(),
    close: () => window.electronAPI?.window.close(),
  },
  dialog: {
    showOpen: (options: any) => window.electronAPI?.dialog.showOpen(options),
    showSave: (options: any) => window.electronAPI?.dialog.showSave(options),
  },
  storage: {
    get: (key: string) => window.electronAPI?.storage.get(key),
    set: (key: string, value: any) => window.electronAPI?.storage.set(key, value),
  },
};
```

```typescript
// components/desktop/title-bar.tsx
import { electronAPI } from '@/lib/desktop';

export function TitleBar() {
  return (
    <div className="title-bar">
      <div className="title-bar-title">KeepTrack</div>
      
      <div className="title-bar-controls">
        <button onClick={() => electronAPI.window.minimize()}>−</button>
        <button onClick={() => electronAPI.window.maximize()}>□</button>
        <button onClick={() => electronAPI.window.close()}>×</button>
      </div>
    </div>
  );
}
```

### 4. Component Usage Example

```typescript
// app/dashboard/assets/page.tsx
import { useAssets } from '@/lib/v1/assets/assets.hooks';
import { Button } from '@/components/ui';
import { useRouter } from 'next/navigation';

export default function AssetsPage() {
  const { assets, loading, error } = useAssets();
  const router = useRouter();

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <div className="header">
        <h1>Assets</h1>
        <Button onClick={() => router.push('/dashboard/assets/create')}>
          Create Asset
        </Button>
      </div>

      <div className="assets-grid">
        {assets.map((asset) => (
          <AssetCard key={asset.id} asset={asset} />
        ))}
      </div>
    </div>
  );
}
```

## File Naming Conventions

### 1. Components
- **kebab-case**: `title-bar.tsx`, `asset-card.tsx`
- Co-located with component: `asset-card.module.css`

### 2. API Files
- **kebab-case.suffix**: 
  - `assets.service.ts` (API calls)
  - `assets.hooks.ts` (React hooks)
  - `assets.types.ts` (TypeScript types)

### 3. Utilities
- **kebab-case**: `format.ts`, `date.ts`, `validation.ts`

### 4. Hooks
- **use-kebab-case**: `use-debounce.ts`, `use-local-storage.ts`

## Import Patterns

```typescript
// API v1
import { useAssets } from '@/lib/v1/assets/assets.hooks';
import { assetsService } from '@/lib/v1/assets/assets.service';
import type { Asset } from '@/lib/v1/assets/assets.types';

// API v2 (future)
import { useAssets } from '@/lib/v2/assets/assets.hooks';

// Components
import { Button, Input, Card } from '@/components/ui';
import { TitleBar } from '@/components/desktop';

// Theme
import { useTheme } from '@/themes';

// Desktop
import { electronAPI } from '@/lib/desktop';

// Utils
import { formatDate, formatNumber } from '@/lib/utils';
```

## API Version Migration

When moving from v1 to v2:

```typescript
// Before (v1)
import { useAssets } from '@/lib/v1/assets/assets.hooks';

// After (v2) - just change the version
import { useAssets } from '@/lib/v2/assets/assets.hooks';

// Both versions can coexist during migration
import { useAssets as useAssetsV1 } from '@/lib/v1/assets/assets.hooks';
import { useAssets as useAssetsV2 } from '@/lib/v2/assets/assets.hooks';
```

## Environment Configuration

```bash
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_API_VERSION=v1
NEXT_PUBLIC_APP_NAME=KeepTrack
```

```typescript
// config/env.ts
export const env = {
  apiUrl: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8080',
  apiVersion: process.env.NEXT_PUBLIC_API_VERSION || 'v1',
  appName: process.env.NEXT_PUBLIC_APP_NAME || 'KeepTrack',
  isDev: process.env.NODE_ENV === 'development',
  isProd: process.env.NODE_ENV === 'production',
};
```

## Shared with Web Application

The following can be **identical** between web and desktop:

### Fully Shared (Link or Copy)
- `components/ui/` - All base components
- `components/forms/` - Form components
- `components/layout/` (except title-bar)
- `lib/v1/`, `lib/v2/` - All API logic
- `lib/utils/` - Utilities
- `lib/hooks/` - Shared hooks
- `themes/` - Entire theme system
- `types/` - Type definitions

### Desktop-Specific Only
- `components/desktop/` - Desktop components
- `lib/desktop/` - Electron wrappers
- `electron/` - Electron main process

## Summary

This structure provides:

1. ✅ **Clean Organization**: Easy to understand
2. ✅ **Version Management**: Clear v1, v2 separation
3. ✅ **Shared Theme**: Common theme system
4. ✅ **No Redundancy**: No duplicate `api/` directory
5. ✅ **API-Focused**: Version number = API version
6. ✅ **Simple Migration**: Easy to switch between versions
7. ✅ **Maximum Sharing**: 90% code shared with web
