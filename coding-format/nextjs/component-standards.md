# Next.js Component Standards

## Component Architecture

### 1. Component Categories

#### Business Components (`/app/[module]/components/`)
- **Purpose**: Business-specific components
- **Examples**: AssetCard, UserProfile, DashboardStats
- **Characteristics**:
  - Contains business logic
  - Module-specific
  - May use shared components

#### Layout Components (`/components/layout/`)
- **Purpose**: Page structure and navigation
- **Examples**: Header, Sidebar, Footer, Navigation
- **Characteristics**:
  - Shared across multiple pages
  - Handles routing and navigation
  - May contain authentication logic

#### Shared Components (`/components/shared/`)
- **Purpose**: Reusable components across modules
- **Examples**: DataTable, FormField, Modal
- **Characteristics**:
  - No business logic
  - Highly reusable
  - Consistent API

### 2. Component Structure Template

```typescript
// ComponentName.tsx
'use client'; // Only when client-side features are needed

import React from 'react';
import { ComponentNameProps } from './ComponentName.types';

export const ComponentName: React.FC<ComponentNameProps> = ({
  variant = 'default',
  size = 'md',
  className,
  children,
  ...props
}) => {
  // Hooks
  const [isLoading, setIsLoading] = useState(false);
  
  // State
  // Effects
  // Handlers
  const handleClick = () => {
    // Handler logic
  };
  
  // Render
  return (
    <div 
      className={className}
      {...props}
    >
      {children}
    </div>
  );
};

ComponentName.displayName = 'ComponentName';

export default ComponentName;
```

### 3. Component Types Template

```typescript
// ComponentName.types.ts
import { ReactNode } from 'react';

export interface ComponentNameProps {
  // Required props
  title: string;
  
  // Optional props with defaults
  variant?: 'primary' | 'secondary' | 'tertiary';
  size?: 'sm' | 'md' | 'lg' | 'xl';
  disabled?: boolean;
  
  // Event handlers
  onClick?: () => void;
  onHover?: () => void;
  
  // Children
  children?: ReactNode;
  
  // Styling
  className?: string;
  
  // Accessibility
  'aria-label'?: string;
  'aria-describedby'?: string;
}

export interface ComponentNameState {
  isLoading: boolean;
  error: string | null;
  data: any;
}

export interface ComponentNameRef {
  focus: () => void;
  blur: () => void;
  getValue: () => string;
}
```

### 4. Component Styles Template

```typescript
// ComponentName.styles.ts
import { cn } from '@/lib/utils';

export const componentStyles = {
  // Base styles
  container: 'relative inline-flex items-center justify-center',
  
  // Variants
  variants: {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
    tertiary: 'bg-transparent border border-gray-300 hover:bg-gray-50',
  },
  
  // Sizes
  sizes: {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
    xl: 'px-8 py-4 text-xl',
  },
  
  // States
  states: {
    disabled: 'opacity-50 cursor-not-allowed',
    loading: 'cursor-wait',
    error: 'border-red-500 focus:ring-red-500',
  },
};

export const getVariantStyles = (variant: string) => {
  return componentStyles.variants[variant] || componentStyles.variants.primary;
};

export const getSizeStyles = (size: string) => {
  return componentStyles.sizes[size] || componentStyles.sizes.md;
};
```

## Component Patterns

### 1. Compound Components

```typescript
// Card.tsx
export const Card = ({ children, className, ...props }) => (
  <div className={cn('bg-white rounded-lg shadow', className)} {...props}>
    {children}
  </div>
);

export const CardHeader = ({ children, className, ...props }) => (
  <div className={cn('px-6 py-4 border-b', className)} {...props}>
    {children}
  </div>
);

export const CardContent = ({ children, className, ...props }) => (
  <div className={cn('px-6 py-4', className)} {...props}>
    {children}
  </div>
);

export const CardFooter = ({ children, className, ...props }) => (
  <div className={cn('px-6 py-4 border-t', className)} {...props}>
    {children}
  </div>
);

// Usage
<Card>
  <CardHeader>
    <h3>Card Title</h3>
  </CardHeader>
  <CardContent>
    <p>Card content</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

### 2. Render Props Pattern

```typescript
// DataProvider.tsx
interface DataProviderProps {
  children: (data: any, loading: boolean, error: string | null) => ReactNode;
  endpoint: string;
}

export const DataProvider: React.FC<DataProviderProps> = ({ children, endpoint }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchData();
  }, [endpoint]);

  return <>{children(data, loading, error)}</>;
};

// Usage
<DataProvider endpoint="/api/assets">
  {(data, loading, error) => (
    loading ? <Loading /> : 
    error ? <Error message={error} /> : 
    <AssetList data={data} />
  )}
</DataProvider>
```

### 3. Higher-Order Components

```typescript
// withAuth.tsx
export const withAuth = <P extends object>(
  Component: React.ComponentType<P>
) => {
  const AuthenticatedComponent: React.FC<P> = (props) => {
    const { user, loading } = useAuth();
    
    if (loading) return <Loading />;
    if (!user) return <LoginRedirect />;
    
    return <Component {...props} />;
  };
  
  AuthenticatedComponent.displayName = `withAuth(${Component.displayName})`;
  return AuthenticatedComponent;
};

// Usage
const ProtectedPage = withAuth(MyPage);
```

## Component Best Practices

### 1. Naming Conventions
- **Components**: PascalCase (e.g., `UserProfile`)
- **Files**: PascalCase (e.g., `UserProfile.tsx`)
- **Props**: camelCase (e.g., `isLoading`, `onClick`)
- **Events**: camelCase with `on` prefix (e.g., `onClick`, `onSubmit`)

### 2. Props Design
```typescript
// Good: Clear, specific props
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  onClick?: () => void;
}

// Bad: Vague, generic props
interface ButtonProps {
  style?: string;
  className?: string;
  props?: any;
}
```

### 3. State Management
```typescript
// Good: Local state for component-specific data
const [isOpen, setIsOpen] = useState(false);
const [selectedItem, setSelectedItem] = useState(null);

// Good: Custom hooks for complex state
const { data, loading, error, refetch } = useAssets();

// Bad: Global state for component-specific data
const [globalModalState, setGlobalModalState] = useState({});
```

### 4. Event Handling
```typescript
// Good: Specific, descriptive handlers
const handleSubmit = (e: FormEvent) => {
  e.preventDefault();
  onSubmit(formData);
};

const handleItemSelect = (item: Item) => {
  setSelectedItem(item);
  onItemSelect?.(item);
};

// Bad: Generic, unclear handlers
const handleClick = (e: any) => {
  // Unclear what this does
};
```

### 5. Accessibility
```typescript
// Good: Proper ARIA attributes
<button
  aria-label="Close modal"
  aria-expanded={isOpen}
  aria-controls="modal-content"
  onClick={handleClose}
>
  <CloseIcon />
</button>

// Good: Semantic HTML
<nav role="navigation" aria-label="Main navigation">
  <ul>
    <li><a href="/dashboard">Dashboard</a></li>
  </ul>
</nav>
```

### 6. Performance Optimization
```typescript
// Good: Memoization for expensive components
const ExpensiveComponent = React.memo(({ data }) => {
  const processedData = useMemo(() => {
    return data.map(item => processItem(item));
  }, [data]);
  
  return <div>{processedData}</div>;
});

// Good: Callback memoization
const handleClick = useCallback((id: string) => {
  onItemClick(id);
}, [onItemClick]);
```

## Component Testing

### 1. Unit Testing
```typescript
// ComponentName.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ComponentName } from './ComponentName';

describe('ComponentName', () => {
  it('renders with correct text', () => {
    render(<ComponentName title="Test Title" />);
    expect(screen.getByText('Test Title')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<ComponentName onClick={handleClick} />);
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### 2. Integration Testing
```typescript
// ComponentName.integration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { ComponentName } from './ComponentName';

describe('ComponentName Integration', () => {
  it('loads data and displays it', async () => {
    render(<ComponentName endpoint="/api/data" />);
    
    await waitFor(() => {
      expect(screen.getByText('Loading...')).toBeInTheDocument();
    });
    
    await waitFor(() => {
      expect(screen.getByText('Data loaded')).toBeInTheDocument();
    });
  });
});
```

This component standards guide ensures consistent, maintainable, and scalable component development in Next.js applications.
