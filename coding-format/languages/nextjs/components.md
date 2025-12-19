# Components Directory Structure Standards

## Overview

The `components/` directory contains all reusable UI components. This document defines the standard directory structure for organizing React/Next.js components using a **Feature-Based Structure**.

---

## Feature-Based Structure (Simple & Practical)

### Complete Directory Structure

```
components/
├── ui/                                 # Base UI components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.types.ts
│   │   ├── Button.styles.ts
│   │   ├── Button.test.tsx             # Optional: tests
│   │   └── index.ts
│   ├── Input/
│   │   ├── Input.tsx
│   │   ├── Input.types.ts
│   │   └── index.ts
│   ├── Card/
│   │   ├── Card.tsx
│   │   └── index.ts
│   └── index.ts                        # Barrel export
├── forms/                              # Form components
│   ├── FormField/
│   │   ├── FormField.tsx
│   │   ├── FormField.types.ts
│   │   └── index.ts
│   ├── FormGroup/
│   │   └── FormGroup.tsx
│   └── index.ts
├── layout/                             # Layout components
│   ├── Header/
│   │   ├── Header.tsx
│   │   ├── Header.types.ts
│   │   └── index.ts
│   ├── Sidebar/
│   │   ├── Sidebar.tsx
│   │   └── index.ts
│   ├── Footer/
│   │   └── Footer.tsx
│   └── index.ts
├── features/                           # Feature-specific components
│   ├── auth/
│   │   ├── LoginForm/
│   │   │   ├── LoginForm.tsx
│   │   │   └── index.ts
│   │   ├── RegisterForm/
│   │   │   └── RegisterForm.tsx
│   │   └── index.ts
│   ├── dashboard/
│   │   ├── AssetCard/
│   │   │   └── AssetCard.tsx
│   │   ├── AssetList/
│   │   │   └── AssetList.tsx
│   │   └── index.ts
│   └── index.ts
└── index.ts                            # Root barrel export
```

### Component Categories

1. **ui/**: Base UI primitives shared across the application (Button, Input, Badge, Avatar, etc.)
2. **forms/**: Form-specific components (FormField, FormGroup, FormValidation, etc.)
3. **layout/**: Layout components (Header, Sidebar, Footer, Container, etc.)
4. **features/**: Feature-specific components organized by domain (auth, dashboard, console, etc.)

**Pros:**
- ✅ Simple and intuitive
- ✅ Easy to find components
- ✅ Scales well
- ✅ Clear categorization
- ✅ Less nesting
- ✅ Feature components are co-located

**Cons:**
- ❌ Can duplicate UI components across features (mitigate by using ui/ for shared components)
- ❌ Requires discipline to keep shared components in ui/ vs features/
- ❌ Feature components might mix with shared (use clear guidelines)

---

## Component File Structure Templates

### 1. Simple Component (No Styles File)

**File**: `components/ui/Button/Button.tsx`

```typescript
'use client'; // Only if needed for client-side features

import React from 'react';
import { ButtonProps } from './Button.types';
import { cn } from '@/lib/utils'; // Utility for className merging

/**
 * Button Component
 * A reusable button component with multiple variants
 */
export const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  className,
  disabled,
  onClick,
  type = 'button',
  ...props
}) => {
  const baseStyles = 'inline-flex items-center justify-center font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
  };
  
  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };

  return (
    <button
      type={type}
      className={cn(
        baseStyles,
        variants[variant],
        sizes[size],
        disabled && 'opacity-50 cursor-not-allowed',
        className
      )}
      disabled={disabled}
      onClick={onClick}
      {...props}
    >
      {children}
    </button>
  );
};

export default Button;
```

**File**: `components/ui/Button/Button.types.ts`

```typescript
import { ButtonHTMLAttributes, ReactNode } from 'react';

/**
 * Button Component Props
 */
export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  /**
   * Button content
   */
  children: ReactNode;
  
  /**
   * Button variant style
   * @default 'primary'
   */
  variant?: 'primary' | 'secondary' | 'danger';
  
  /**
   * Button size
   * @default 'md'
   */
  size?: 'sm' | 'md' | 'lg';
  
  /**
   * Whether button is disabled
   */
  disabled?: boolean;
  
  /**
   * Click handler
   */
  onClick?: () => void;
}
```

**File**: `components/ui/Button/index.ts`

```typescript
export { Button } from './Button';
export type { ButtonProps } from './Button.types';
export default Button;
```

### 2. Component with Styles File

**File**: `components/ui/Button/Button.styles.ts`

```typescript
import { cn } from '@/lib/utils';

export const buttonStyles = {
  base: 'inline-flex items-center justify-center font-medium rounded-lg transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2',
  
  variants: {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
  },
  
  sizes: {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  },
  
  disabled: 'opacity-50 cursor-not-allowed',
};

export const getButtonStyles = (
  variant: keyof typeof buttonStyles.variants,
  size: keyof typeof buttonStyles.sizes,
  disabled?: boolean,
  className?: string
) => {
  return cn(
    buttonStyles.base,
    buttonStyles.variants[variant],
    buttonStyles.sizes[size],
    disabled && buttonStyles.disabled,
    className
  );
};
```

**File**: `components/ui/Button/Button.tsx` (using styles)

```typescript
'use client';

import React from 'react';
import { ButtonProps } from './Button.types';
import { getButtonStyles } from './Button.styles';

export const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  className,
  disabled,
  onClick,
  type = 'button',
  ...props
}) => {
  return (
    <button
      type={type}
      className={getButtonStyles(variant, size, disabled, className)}
      disabled={disabled}
      onClick={onClick}
      {...props}
    >
      {children}
    </button>
  );
};

export default Button;
```

### 3. Complex Component with Sub-components

**File**: `components/forms/FormField/FormField.tsx`

```typescript
'use client';

import React from 'react';
import { FormFieldProps } from './FormField.types';
import { Label } from './Label';
import { Input } from './Input';
import { Error } from './Error';

/**
 * FormField Component
 * A complete form field with label, input, and error handling
 */
export const FormField: React.FC<FormFieldProps> = ({
  label,
  name,
  error,
  required,
  children,
  className,
}) => {
  return (
    <div className={className}>
      {label && (
        <Label htmlFor={name} required={required}>
          {label}
        </Label>
      )}
      {children || <Input name={name} error={!!error} />}
      {error && <Error message={error} />}
    </div>
  );
};

// Export sub-components
FormField.Label = Label;
FormField.Input = Input;
FormField.Error = Error;

export default FormField;
```

**File**: `components/forms/FormField/FormField.types.ts`

```typescript
import { ReactNode } from 'react';

export interface FormFieldProps {
  /**
   * Field label
   */
  label?: string;
  
  /**
   * Field name (for form association)
   */
  name: string;
  
  /**
   * Error message to display
   */
  error?: string;
  
  /**
   * Whether field is required
   */
  required?: boolean;
  
  /**
   * Custom input component
   */
  children?: ReactNode;
  
  /**
   * Additional className
   */
  className?: string;
}
```

### 4. Feature-Specific Component

**File**: `components/features/auth/LoginForm/LoginForm.tsx`

```typescript
'use client';

import React, { useState } from 'react';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { FormField } from '@/components/forms/FormField';

/**
 * LoginForm Component
 * Feature-specific login form for authentication
 */
export const LoginForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Handle login logic
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <FormField label="Email" name="email" required>
        <Input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </FormField>
      
      <FormField label="Password" name="password" required>
        <Input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </FormField>
      
      <Button type="submit" variant="primary">
        Login
      </Button>
    </form>
  );
};

export default LoginForm;
```

**File**: `components/features/auth/LoginForm/index.ts`

```typescript
export { LoginForm } from './LoginForm';
export default LoginForm;
```

---

## Naming Conventions

### Files
- **Component**: `[ComponentName].tsx` (PascalCase, e.g., `Button.tsx`)
- **Types**: `[ComponentName].types.ts` (e.g., `Button.types.ts`)
- **Styles**: `[ComponentName].styles.ts` (e.g., `Button.styles.ts`)
- **Tests**: `[ComponentName].test.tsx` (e.g., `Button.test.tsx`)
- **Index**: Always `index.ts`

### Components
- **Component Name**: PascalCase (e.g., `Button`, `FormField`)
- **Props Interface**: `[ComponentName]Props` (e.g., `ButtonProps`)
- **Sub-components**: Attach to main component (e.g., `FormField.Label`)

### Directories
- **Component Directory**: PascalCase (e.g., `Button/`, `FormField/`)
- **Category Directory**: kebab-case (e.g., `ui/`, `forms/`, `features/`)

---

## Import Patterns

### Internal Imports (within component)
```typescript
// In Button.tsx
import { ButtonProps } from './Button.types';
import { getButtonStyles } from './Button.styles';
```

### External Imports (from other components)
```typescript
// From lib
import { cn } from '@/lib/utils';

// From shared UI components
import { Input } from '@/components/ui/Input';
import { Button } from '@/components/ui/Button';

// From form components
import { FormField } from '@/components/forms/FormField';

// From feature components
import { LoginForm } from '@/components/features/auth/LoginForm';
```

### Public API Exports
```typescript
// From component index
import { Button } from '@/components/ui/Button';

// From category index
import { Button, Input } from '@/components/ui';

// From root index
import { Button } from '@/components';
```

---

## Barrel Exports

### Component Index
**File**: `components/ui/Button/index.ts`

```typescript
export { Button } from './Button';
export type { ButtonProps } from './Button.types';
export { buttonStyles, getButtonStyles } from './Button.styles';
export default Button;
```

### Category Index
**File**: `components/ui/index.ts`

```typescript
export { Button } from './Button';
export { Input } from './Input';
export { Card } from './Card';
// ... other UI components
```

**File**: `components/features/auth/index.ts`

```typescript
export { LoginForm } from './LoginForm';
export { RegisterForm } from './RegisterForm';
// ... other auth components
```

### Root Index
**File**: `components/index.ts`

```typescript
// UI Components
export * from './ui';

// Forms
export * from './forms';

// Layout
export * from './layout';

// Features
export * from './features';
```

---

## Best Practices

### 1. Component Structure
- One component per file
- Co-locate related files (types, styles, tests)
- Use index.ts for clean exports
- Keep components focused and small

### 2. Type Safety
- Always define TypeScript types
- Export types from `.types.ts` files
- Use `type` imports: `import type { ... }`
- Document props with JSDoc

### 3. Styling
- Use Tailwind CSS classes
- Extract complex styles to `.styles.ts`
- Use `cn()` utility for className merging
- Follow design system tokens

### 4. Client Components
- Use `'use client'` only when needed
- Prefer Server Components by default
- Mark interactive components as client

### 5. Composition
- Build complex components from simple ones
- Use compound components pattern when needed
- Prefer composition over props drilling

### 6. Reusability
- Make components generic and reusable
- Accept className for customization
- Use variants for different styles
- Provide sensible defaults

### 7. Feature vs Shared Components
- **Shared components** → `ui/`, `forms/`, `layout/`
- **Feature-specific components** → `features/[feature-name]/`
- If a component is used in 2+ features, move it to `ui/` or appropriate category
- Keep feature components focused on their domain

### 8. Documentation
- Document all props with JSDoc
- Include usage examples in comments
- Explain complex logic
- Add component description

---

## Component Categories Guide

### When to use each category:

- **ui/**: Base building blocks used across multiple features (Button, Input, Badge, Avatar, Card, etc.)
- **forms/**: Form-specific components (FormField, FormGroup, FormValidation, etc.)
- **layout/**: Page structure components (Header, Sidebar, Footer, Container, etc.)
- **features/**: Feature-specific components organized by domain:
  - `features/auth/` - Authentication components (LoginForm, RegisterForm, etc.)
  - `features/dashboard/` - Dashboard-specific components (AssetCard, AssetList, etc.)
  - `features/console/` - Console-specific components (UserTable, OrganizationCard, etc.)

---

## Complete Example: Button Component

```
components/ui/Button/
├── Button.tsx              # Main component
├── Button.types.ts         # TypeScript types
├── Button.styles.ts        # Style utilities
├── Button.test.tsx         # Tests (optional)
└── index.ts               # Barrel export
```

**Usage:**
```typescript
import { Button } from '@/components/ui/Button';

<Button variant="primary" size="md" onClick={handleClick}>
  Click Me
</Button>
```

---

## Migration Guide

When creating new components:

1. **Determine if component is shared or feature-specific**
   - Used in 2+ features? → `ui/`, `forms/`, or `layout/`
   - Feature-specific? → `features/[feature-name]/`

2. **Create component directory**: `components/[category]/[ComponentName]/`

3. **Create component file**: `[ComponentName].tsx`

4. **Create types file**: `[ComponentName].types.ts`

5. **Create styles file (if needed)**: `[ComponentName].styles.ts`

6. **Create index.ts** with exports

7. **Update category index.ts**

8. **Update root index.ts** (if needed)

9. **Document component and props**

10. **Add tests** (optional)

This structure ensures scalability, maintainability, and follows industry best practices for React/Next.js components while keeping the organization simple and intuitive.
