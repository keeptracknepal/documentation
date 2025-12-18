# React Native Mobile Coding Format - Summary

## Overview
This document defines the industry-standard coding format for React Native mobile projects. The format emphasizes clean MVC architecture, scalability, maintainability, and follows React Native best practices with a simple, organized directory structure.

## Core Principles

### 1. MVC Architecture
- **Model**: Data structures, business entities, and state management
- **View**: UI components and screens
- **Controller**: Business logic, API calls, and data flow

### 2. Layered Architecture
```
┌─────────────────┐
│     Screens     │ ← Presentation Layer (UI)
├─────────────────┤
│   Components    │ ← Reusable UI Components
├─────────────────┤
│    Services     │ ← Business Logic Layer
├─────────────────┤
│      API        │ ← Data Access Layer
├─────────────────┤
│     Models      │ ← Data Structures
├─────────────────┤
│      Types      │ ← TypeScript Definitions
├─────────────────┤
│     Utils       │ ← Utility Functions
└─────────────────┘
```

### 3. Separation of Concerns
- **Single Responsibility**: Each component has one reason to change
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Interface Segregation**: Small, focused interfaces
- **Composition over Inheritance**: Use composition for component relationships

## Directory Structure

```
mobile/
├── src/                              # Source code
│   ├── components/                   # Reusable UI components
│   │   ├── common/                   # Common components
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Button.types.ts
│   │   │   │   ├── Button.styles.ts
│   │   │   │   └── index.ts
│   │   │   ├── Input/
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Input.types.ts
│   │   │   │   ├── Input.styles.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   ├── forms/                    # Form-specific components
│   │   │   ├── LoginForm/
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   ├── LoginForm.types.ts
│   │   │   │   ├── LoginForm.styles.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── screens/                      # Screen components
│   │   ├── auth/                     # Authentication screens
│   │   │   ├── LoginScreen/
│   │   │   │   ├── LoginScreen.tsx
│   │   │   │   ├── LoginScreen.types.ts
│   │   │   │   ├── LoginScreen.styles.ts
│   │   │   │   ├── LoginScreen.hooks.ts
│   │   │   │   └── index.ts
│   │   │   ├── RegisterScreen/
│   │   │   │   ├── RegisterScreen.tsx
│   │   │   │   ├── RegisterScreen.types.ts
│   │   │   │   ├── RegisterScreen.styles.ts
│   │   │   │   ├── RegisterScreen.hooks.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   ├── dashboard/                # Dashboard screens
│   │   │   ├── DashboardScreen/
│   │   │   │   ├── DashboardScreen.tsx
│   │   │   │   ├── DashboardScreen.types.ts
│   │   │   │   ├── DashboardScreen.styles.ts
│   │   │   │   ├── DashboardScreen.hooks.ts
│   │   │   │   └── index.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── navigation/                   # Navigation configuration
│   │   ├── AppNavigator.tsx
│   │   ├── AuthNavigator.tsx
│   │   ├── MainNavigator.tsx
│   │   ├── navigation.types.ts
│   │   └── index.ts
│   ├── services/                     # Business logic services
│   │   ├── auth/                     # Authentication service
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.types.ts
│   │   │   ├── auth.utils.ts
│   │   │   └── index.ts
│   │   ├── api/                      # API services
│   │   │   ├── api.client.ts
│   │   │   ├── api.types.ts
│   │   │   ├── api.utils.ts
│   │   │   └── index.ts
│   │   ├── storage/                  # Storage services
│   │   │   ├── storage.service.ts
│   │   │   ├── storage.types.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── hooks/                        # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useApi.ts
│   │   ├── useStorage.ts
│   │   ├── useDebounce.ts
│   │   └── index.ts
│   ├── utils/                        # Utility functions
│   │   ├── validation.ts
│   │   ├── formatting.ts
│   │   ├── constants.ts
│   │   ├── helpers.ts
│   │   └── index.ts
│   ├── types/                        # TypeScript type definitions
│   │   ├── api.types.ts
│   │   ├── auth.types.ts
│   │   ├── navigation.types.ts
│   │   ├── common.types.ts
│   │   └── index.ts
│   ├── theme/                        # Theme and styling
│   │   ├── colors.ts
│   │   ├── typography.ts
│   │   ├── spacing.ts
│   │   ├── components.ts
│   │   └── index.ts
│   ├── lib/                          # External library configurations
│   │   ├── config/                   # App configuration
│   │   │   ├── index.ts
│   │   │   └── constants.ts
│   │   ├── storage/                  # Storage configuration
│   │   │   ├── index.ts
│   │   │   └── types.ts
│   │   └── index.ts
│   ├── assets/                       # Static assets
│   │   ├── images/
│   │   ├── icons/
│   │   ├── fonts/
│   │   └── index.ts
│   └── index.ts                      # Main entry point
├── android/                          # Android-specific code
├── ios/                              # iOS-specific code
├── web/                              # Web-specific code (for React Native Web)
├── public/                           # Public assets
├── package.json
├── tsconfig.json
├── babel.config.js
├── metro.config.js
└── README.md
```

## File Naming Conventions

### 1. Component Files
```
ComponentName/
├── ComponentName.tsx          # Main component
├── ComponentName.types.ts     # TypeScript types
├── ComponentName.styles.ts    # Styling (if complex)
├── ComponentName.hooks.ts     # Custom hooks (if needed)
├── ComponentName.test.tsx     # Tests
└── index.ts                   # Export file
```

### 2. Service Files
```
serviceName/
├── serviceName.service.ts     # Main service logic
├── serviceName.types.ts       # TypeScript types
├── serviceName.utils.ts       # Utility functions
├── serviceName.test.ts        # Tests
└── index.ts                   # Export file
```

### 3. Screen Files
```
ScreenName/
├── ScreenName.tsx             # Main screen component
├── ScreenName.types.ts        # TypeScript types
├── ScreenName.styles.ts       # Styling
├── ScreenName.hooks.ts        # Custom hooks
├── ScreenName.test.tsx        # Tests
└── index.ts                   # Export file
```

## Code Organization Rules

### 1. Component Structure
```typescript
// ComponentName.tsx
import React from 'react';
import { View, Text } from 'react-native';
import { ComponentNameProps } from './ComponentName.types';
import { styles } from './ComponentName.styles';

export const ComponentName: React.FC<ComponentNameProps> = ({
  prop1,
  prop2,
  onPress,
}) => {
  // Hooks
  const [state, setState] = useState();
  
  // Event handlers
  const handlePress = () => {
    onPress?.();
  };
  
  // Render
  return (
    <View style={styles.container}>
      <Text style={styles.text}>{prop1}</Text>
    </View>
  );
};
```

### 2. Service Structure
```typescript
// serviceName.service.ts
import { ServiceNameTypes } from './serviceName.types';
import { apiClient } from '../api';

export class ServiceNameService {
  // Public methods
  static async getData(): Promise<ServiceNameTypes.Data> {
    try {
      const response = await apiClient.get('/endpoint');
      return response.data;
    } catch (error) {
      throw new Error('Failed to get data');
    }
  }
  
  // Private methods
  private static validateData(data: any): boolean {
    return data && typeof data === 'object';
  }
}
```

### 3. Hook Structure
```typescript
// useHookName.ts
import { useState, useEffect } from 'react';
import { HookNameTypes } from './types';

export const useHookName = (param: string) => {
  const [state, setState] = useState<HookNameTypes.State>({
    loading: false,
    data: null,
    error: null,
  });
  
  useEffect(() => {
    // Effect logic
  }, [param]);
  
  const method = () => {
    // Method logic
  };
  
  return {
    state,
    method,
  };
};
```

## Import/Export Rules

### 1. Import Order
```typescript
// 1. React and React Native imports
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';

// 2. Third-party library imports
import { NavigationProp } from '@react-navigation/native';
import AsyncStorage from '@react-native-async-storage/async-storage';

// 3. Internal imports (absolute paths)
import { Button } from '@/components';
import { useAuth } from '@/hooks';
import { authService } from '@/services';

// 4. Relative imports
import { ComponentProps } from './Component.types';
import { styles } from './Component.styles';
```

### 2. Export Patterns
```typescript
// index.ts files
export { ComponentName } from './ComponentName';
export type { ComponentNameProps } from './ComponentName.types';

// Main component files
export const ComponentName: React.FC<ComponentNameProps> = () => {
  // Component implementation
};

// Service files
export class ServiceName {
  // Service implementation
}

// Hook files
export const useHookName = () => {
  // Hook implementation
};
```

## Styling Guidelines

### 1. StyleSheet Usage
```typescript
// ComponentName.styles.ts
import { StyleSheet } from 'react-native';
import { colors, spacing, typography } from '@/theme';

export const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.background,
    padding: spacing.md,
  },
  title: {
    ...typography.h1,
    color: colors.text,
    marginBottom: spacing.lg,
  },
  button: {
    backgroundColor: colors.primary,
    padding: spacing.md,
    borderRadius: 8,
  },
});
```

### 2. Theme Integration
```typescript
// Always use theme values
import { colors, spacing, typography } from '@/theme';

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.background,    // ✅ Good
    padding: spacing.md,                   // ✅ Good
    fontSize: typography.body.fontSize,    // ✅ Good
  },
  // Avoid hardcoded values
  badContainer: {
    backgroundColor: '#ffffff',            // ❌ Bad
    padding: 16,                          // ❌ Bad
    fontSize: 14,                         // ❌ Bad
  },
});
```

## State Management

### 1. Local State
```typescript
// Use useState for simple local state
const [loading, setLoading] = useState(false);
const [data, setData] = useState<DataType | null>(null);
```

### 2. Global State
```typescript
// Use Context for global state
const AuthContext = createContext<AuthContextType | null>(null);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({
  children,
}) => {
  const [user, setUser] = useState<User | null>(null);
  
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### 3. Custom Hooks
```typescript
// Extract complex state logic into custom hooks
export const useAuth = () => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(false);
  
  const login = async (credentials: LoginCredentials) => {
    setLoading(true);
    try {
      const user = await authService.login(credentials);
      setUser(user);
    } catch (error) {
      throw error;
    } finally {
      setLoading(false);
    }
  };
  
  return { user, loading, login };
};
```

## Error Handling

### 1. API Error Handling
```typescript
// serviceName.service.ts
export class ServiceName {
  static async getData(): Promise<DataType> {
    try {
      const response = await apiClient.get('/endpoint');
      return response.data;
    } catch (error) {
      if (error.response?.status === 401) {
        throw new Error('Unauthorized');
      }
      if (error.response?.status === 404) {
        throw new Error('Data not found');
      }
      throw new Error('Failed to fetch data');
    }
  }
}
```

### 2. Component Error Handling
```typescript
// ComponentName.tsx
export const ComponentName: React.FC<ComponentNameProps> = () => {
  const [error, setError] = useState<string | null>(null);
  
  const handleAction = async () => {
    try {
      setError(null);
      await someAsyncAction();
    } catch (error) {
      setError(error.message);
    }
  };
  
  return (
    <View>
      {error && <Text style={styles.error}>{error}</Text>}
      {/* Rest of component */}
    </View>
  );
};
```

## Testing Guidelines

### 1. Component Testing
```typescript
// ComponentName.test.tsx
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  it('renders correctly', () => {
    const { getByText } = render(<ComponentName title="Test" />);
    expect(getByText('Test')).toBeTruthy();
  });
  
  it('handles press events', () => {
    const onPress = jest.fn();
    const { getByText } = render(
      <ComponentName title="Test" onPress={onPress} />
    );
    
    fireEvent.press(getByText('Test'));
    expect(onPress).toHaveBeenCalled();
  });
});
```

### 2. Service Testing
```typescript
// serviceName.test.ts
import { ServiceName } from './serviceName.service';

describe('ServiceName', () => {
  it('fetches data successfully', async () => {
    const data = await ServiceName.getData();
    expect(data).toBeDefined();
  });
  
  it('handles errors correctly', async () => {
    // Mock error scenario
    await expect(ServiceName.getData()).rejects.toThrow();
  });
});
```

## Performance Guidelines

### 1. Component Optimization
```typescript
// Use React.memo for expensive components
export const ExpensiveComponent = React.memo<Props>(({ data }) => {
  return <View>{/* Component content */}</View>;
});

// Use useMemo for expensive calculations
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// Use useCallback for event handlers
const handlePress = useCallback(() => {
  onPress();
}, [onPress]);
```

### 2. List Optimization
```typescript
// Use FlatList for large lists
<FlatList
  data={items}
  renderItem={({ item }) => <ItemComponent item={item} />}
  keyExtractor={(item) => item.id}
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
/>
```

## Security Guidelines

### 1. Data Protection
```typescript
// Never store sensitive data in plain text
import { setItem } from '@/lib/storage';

// ✅ Good - Use secure storage
await setItem('user_token', token, { encrypt: true });

// ❌ Bad - Plain text storage
await AsyncStorage.setItem('password', password);
```

### 2. Input Validation
```typescript
// Always validate user input
import { VALIDATION } from '@/lib/config';

const validateEmail = (email: string): boolean => {
  return VALIDATION.EMAIL_REGEX.test(email);
};

const validatePassword = (password: string): boolean => {
  return password.length >= VALIDATION.PASSWORD_MIN_LENGTH;
};
```

## Best Practices Summary

### 1. Code Organization
- ✅ Use absolute imports with `@/` prefix
- ✅ Group related files in folders
- ✅ Use index.ts files for clean exports
- ✅ Separate types, styles, and logic into different files
- ❌ Don't mix different concerns in single files

### 2. Component Design
- ✅ Keep components small and focused
- ✅ Use TypeScript for type safety
- ✅ Extract reusable logic into custom hooks
- ✅ Use composition over inheritance
- ❌ Don't create overly complex components

### 3. State Management
- ✅ Use local state for component-specific data
- ✅ Use Context for global state
- ✅ Extract complex state logic into custom hooks
- ❌ Don't overuse global state

### 4. Performance
- ✅ Use React.memo for expensive components
- ✅ Use useMemo and useCallback appropriately
- ✅ Optimize lists with FlatList
- ❌ Don't create unnecessary re-renders

### 5. Error Handling
- ✅ Handle errors gracefully
- ✅ Provide meaningful error messages
- ✅ Use try-catch blocks for async operations
- ❌ Don't ignore errors

This format ensures maintainable, scalable, and professional React Native applications following industry best practices.
