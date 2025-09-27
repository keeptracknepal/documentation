# Next.js Service Layer Standards

## Service Architecture Overview

The service layer acts as an abstraction between the UI components and the API, providing a clean interface for data operations and business logic. The structure is organized to support independent dashboard and console modules.

## Service Layer Structure

### 1. HTTP Client Configuration

```typescript
// lib/common/client/client.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';
import { API_BASE_URL } from '@/lib/config';

export interface HttpResponse<T = any> {
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

class HttpClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: API_BASE_URL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        // Add auth token
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response: AxiosResponse) => response,
      (error) => {
        // Handle common errors
        if (error.response?.status === 401) {
          // Redirect to login
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.get<T>(url, config);
    return response.data;
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.post<T>(url, data, config);
    return response.data;
  }

  async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.put<T>(url, data, config);
    return response.data;
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response = await this.client.delete<T>(url, config);
    return response.data;
  }
}

export const httpClient = new HttpClient();
```

### 2. Dashboard Services

#### Assets Service
```typescript
// lib/dashboard/services/assets/assets.service.ts
import { httpClient } from '@/lib/common/client/client';
import { Asset, CreateAssetRequest, UpdateAssetRequest, GetAssetsParams } from './assets.types';

export class AssetsService {
  private baseUrl = '/dashboard/pages/assets';

  async getAssets(params?: GetAssetsParams): Promise<PaginatedResponse<Asset>> {
    const response = await httpClient.get<PaginatedResponse<Asset>>(this.baseUrl, { params });
    return response;
  }

  async getAssetById(id: string): Promise<Asset> {
    const response = await httpClient.get<HttpResponse<Asset>>(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async createAsset(data: CreateAssetRequest): Promise<Asset> {
    const response = await httpClient.post<HttpResponse<Asset>>(`${this.baseUrl}/create`, data);
    return response.data;
  }

  async updateAsset(id: string, data: UpdateAssetRequest): Promise<Asset> {
    const response = await httpClient.put<HttpResponse<Asset>>(`${this.baseUrl}/${id}`, data);
    return response.data;
  }

  async deleteAsset(id: string): Promise<void> {
    await httpClient.delete(`${this.baseUrl}/${id}`);
  }

  async getAssetCategories(): Promise<Category[]> {
    const response = await httpClient.get<HttpResponse<Category[]>>(`${this.baseUrl}/categories`);
    return response.data;
  }

  async getAssetBranches(): Promise<Branch[]> {
    const response = await httpClient.get<HttpResponse<Branch[]>>(`${this.baseUrl}/branches`);
    return response.data;
  }

  async getAssetStats(): Promise<AssetStats> {
    const response = await httpClient.get<HttpResponse<AssetStats>>(`${this.baseUrl}/stats`);
    return response.data;
  }
}

export const assetsService = new AssetsService();
```

#### Employees Service
```typescript
// lib/dashboard/services/employees/employees.service.ts
import { apiClient } from '@/lib/common/api/client';
import { Employee, CreateEmployeeRequest, UpdateEmployeeRequest } from './employees.types';

export class EmployeesService {
  private baseUrl = '/dashboard/pages/employees';

  async getEmployees(): Promise<Employee[]> {
    const response = await apiClient.get<Employee[]>(this.baseUrl);
    return response;
  }

  async getEmployeeById(id: string): Promise<Employee> {
    const response = await apiClient.get<ApiResponse<Employee>>(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async createEmployee(data: CreateEmployeeRequest): Promise<Employee> {
    const response = await apiClient.post<ApiResponse<Employee>>(`${this.baseUrl}/create`, data);
    return response.data;
  }

  async updateEmployee(id: string, data: UpdateEmployeeRequest): Promise<Employee> {
    const response = await apiClient.put<ApiResponse<Employee>>(`${this.baseUrl}/${id}`, data);
    return response.data;
  }

  async deleteEmployee(id: string): Promise<void> {
    await apiClient.delete(`${this.baseUrl}/${id}`);
  }
}

export const employeesService = new EmployeesService();
```

### 3. Console Services

#### Organizations Service
```typescript
// lib/console/services/organizations/organizations.service.ts
import { apiClient } from '@/lib/common/api/client';
import { Organization, CreateOrganizationRequest, UpdateOrganizationRequest } from './organizations.types';

export class OrganizationsService {
  private baseUrl = '/console/organizations';

  async getOrganizations(): Promise<Organization[]> {
    const response = await apiClient.get<Organization[]>(this.baseUrl);
    return response;
  }

  async getOrganizationById(id: string): Promise<Organization> {
    const response = await apiClient.get<ApiResponse<Organization>>(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async createOrganization(data: CreateOrganizationRequest): Promise<Organization> {
    const response = await apiClient.post<ApiResponse<Organization>>(this.baseUrl, data);
    return response.data;
  }

  async updateOrganization(id: string, data: UpdateOrganizationRequest): Promise<Organization> {
    const response = await apiClient.put<ApiResponse<Organization>>(`${this.baseUrl}/${id}`, data);
    return response.data;
  }

  async deleteOrganization(id: string): Promise<void> {
    await apiClient.delete(`${this.baseUrl}/${id}`);
  }
}

export const organizationsService = new OrganizationsService();
```

#### Users Service
```typescript
// lib/console/services/users/users.service.ts
import { apiClient } from '@/lib/common/api/client';
import { User, CreateUserRequest, UpdateUserRequest } from './users.types';

export class UsersService {
  private baseUrl = '/console/users';

  async getUsers(): Promise<User[]> {
    const response = await apiClient.get<User[]>(this.baseUrl);
    return response;
  }

  async getUserById(id: string): Promise<User> {
    const response = await apiClient.get<ApiResponse<User>>(`${this.baseUrl}/${id}`);
    return response.data;
  }

  async createUser(data: CreateUserRequest): Promise<User> {
    const response = await apiClient.post<ApiResponse<User>>(this.baseUrl, data);
    return response.data;
  }

  async updateUser(id: string, data: UpdateUserRequest): Promise<User> {
    const response = await apiClient.put<ApiResponse<User>>(`${this.baseUrl}/${id}`, data);
    return response.data;
  }

  async deleteUser(id: string): Promise<void> {
    await apiClient.delete(`${this.baseUrl}/${id}`);
  }
}

export const usersService = new UsersService();
```

### 4. Service Types

#### Dashboard Types
```typescript
// lib/dashboard/types/assets.ts
export interface Asset {
  id: string;
  name: string;
  model_number?: string;
  manufacturer?: string;
  barcode?: string;
  status: 'active' | 'inactive' | 'maintenance' | 'retired' | 'scrapped';
  purchase_amount?: number;
  created_at: string;
  branch_name: string;
  category_icon?: string;
}

export interface CreateAssetRequest {
  name: string;
  model_number?: string;
  manufacturer?: string;
  barcode?: string;
  status: string;
  purchase_amount?: number;
  branch_id: string;
  category_id?: string;
  organization_id: string;
}

export interface UpdateAssetRequest {
  name?: string;
  model_number?: string;
  manufacturer?: string;
  barcode?: string;
  status?: string;
  purchase_amount?: number;
  branch_id?: string;
  category_id?: string;
}

export interface GetAssetsParams {
  organization_id?: string;
  branch_id?: string;
  category_id?: string;
  status?: string;
  search?: string;
  limit?: number;
  offset?: number;
  sort_by?: string;
  sort_order?: 'asc' | 'desc';
}
```

#### Console Types
```typescript
// lib/console/types/organizations.ts
export interface Organization {
  id: string;
  name: string;
  description?: string;
  is_active: boolean;
  created_at: string;
  updated_at: string;
}

export interface CreateOrganizationRequest {
  name: string;
  description?: string;
}

export interface UpdateOrganizationRequest {
  name?: string;
  description?: string;
  is_active?: boolean;
}
```

### 5. Service Index Files

#### Dashboard Services Index
```typescript
// lib/dashboard/services/index.ts
export { AssetsService, assetsService } from './assets/assets.service';
export { EmployeesService, employeesService } from './employees/employees.service';

export type {
  Asset,
  CreateAssetRequest,
  UpdateAssetRequest,
  GetAssetsParams,
} from './assets/assets.types';

export type {
  Employee,
  CreateEmployeeRequest,
  UpdateEmployeeRequest,
} from './employees/employees.types';
```

#### Console Services Index
```typescript
// lib/console/services/index.ts
export { OrganizationsService, organizationsService } from './organizations/organizations.service';
export { UsersService, usersService } from './users/users.service';

export type {
  Organization,
  CreateOrganizationRequest,
  UpdateOrganizationRequest,
} from './organizations/organizations.types';

export type {
  User,
  CreateUserRequest,
  UpdateUserRequest,
} from './users/users.types';
```

## Custom Hooks for Services

### 1. Dashboard Hooks

#### Assets Hooks
```typescript
// lib/dashboard/hooks/useAssets.ts
import { useState, useEffect, useCallback } from 'react';
import { assetsService } from '@/lib/dashboard/services';
import { Asset, GetAssetsParams } from '@/lib/dashboard/types';

export const useAssets = (params?: GetAssetsParams) => {
  const [assets, setAssets] = useState<Asset[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [pagination, setPagination] = useState({
    total: 0,
    limit: 50,
    offset: 0,
    hasMore: false,
  });

  const fetchAssets = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await assetsService.getAssets(params);
      setAssets(response.data);
      setPagination(response.pagination);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  }, [params]);

  useEffect(() => {
    fetchAssets();
  }, [fetchAssets]);

  const refetch = useCallback(() => {
    fetchAssets();
  }, [fetchAssets]);

  return {
    assets,
    loading,
    error,
    pagination,
    refetch,
  };
};
```

#### Employees Hooks
```typescript
// lib/dashboard/hooks/useEmployees.ts
import { useState, useEffect, useCallback } from 'react';
import { employeesService } from '@/lib/dashboard/services';
import { Employee } from '@/lib/dashboard/types';

export const useEmployees = () => {
  const [employees, setEmployees] = useState<Employee[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchEmployees = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await employeesService.getEmployees();
      setEmployees(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchEmployees();
  }, [fetchEmployees]);

  return {
    employees,
    loading,
    error,
    refetch: fetchEmployees,
  };
};
```

### 2. Console Hooks

#### Organizations Hooks
```typescript
// lib/console/hooks/useOrganizations.ts
import { useState, useEffect, useCallback } from 'react';
import { organizationsService } from '@/lib/console/services';
import { Organization } from '@/lib/console/types';

export const useOrganizations = () => {
  const [organizations, setOrganizations] = useState<Organization[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchOrganizations = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await organizationsService.getOrganizations();
      setOrganizations(data);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    fetchOrganizations();
  }, [fetchOrganizations]);

  return {
    organizations,
    loading,
    error,
    refetch: fetchOrganizations,
  };
};
```

### 3. Shared Hooks

#### Common API Hook
```typescript
// lib/common/hooks/useApi.ts
import { useState, useCallback } from 'react';

export const useApi = <T>() => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const execute = useCallback(async (apiCall: () => Promise<T>) => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiCall();
      setData(result);
      return result;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'An error occurred';
      setError(errorMessage);
      throw new Error(errorMessage);
    } finally {
      setLoading(false);
    }
  }, []);

  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setLoading(false);
  }, []);

  return {
    data,
    loading,
    error,
    execute,
    reset,
  };
};
```

## Error Handling

### 1. Error Types
```typescript
// lib/common/types/errors.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

export class ValidationError extends Error {
  constructor(
    message: string,
    public field: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

export class NetworkError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NetworkError';
  }
}
```

### 2. Error Handling Service
```typescript
// lib/common/utils/errorHandler.ts
import { ApiError, ValidationError, NetworkError } from '../types/errors';

export const handleApiError = (error: any): Error => {
  if (error.response) {
    // Server responded with error status
    const { status, data } = error.response;
    return new ApiError(data.message || 'An error occurred', status, data.code);
  } else if (error.request) {
    // Network error
    return new NetworkError('Network error occurred');
  } else {
    // Other error
    return new Error(error.message || 'An unexpected error occurred');
  }
};

export const isApiError = (error: Error): error is ApiError => {
  return error instanceof ApiError;
};

export const isValidationError = (error: Error): error is ValidationError => {
  return error instanceof ValidationError;
};

export const isNetworkError = (error: Error): error is NetworkError => {
  return error instanceof NetworkError;
};
```

## Module Independence

### 1. Dashboard Module
```typescript
// lib/dashboard/index.ts
export * from './services';
export * from './hooks';
export * from './utils';
export * from './types';

// Dashboard-specific configuration
export const dashboardConfig = {
  apiBaseUrl: '/dashboard',
  features: ['assets', 'employees', 'settings'],
  permissions: ['view', 'create', 'update', 'delete'],
};
```

### 2. Console Module
```typescript
// lib/console/index.ts
export * from './services';
export * from './hooks';
export * from './utils';
export * from './types';

// Console-specific configuration
export const consoleConfig = {
  apiBaseUrl: '/console',
  features: ['organizations', 'users', 'branches', 'positions'],
  permissions: ['admin', 'manage', 'view'],
};
```

### 3. Common Module
```typescript
// lib/common/index.ts
export * from './api';
export * from './hooks';
export * from './utils';
export * from './types';

// Shared configuration
export const commonConfig = {
  apiTimeout: 10000,
  retryAttempts: 3,
  cacheTimeout: 300000, // 5 minutes
};
```

This service layer standards guide ensures independent, maintainable, and scalable service implementation for both dashboard and console modules in Next.js applications.