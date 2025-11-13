# TypeScript Interface Definitions

TypeScript interfaces for props, state, and domain types. Provides compile-time type checking and IDE support.

## Pattern

```tsx
interface ComponentProps {
  required: string;
  optional?: number;
  callback: (value: string) => void;
}
```

## Why Use This Pattern

- **Type safety**: Catch type errors at compile time
- **IDE support**: Autocomplete and inline documentation
- **Self-documenting**: Interface names and types explain structure
- **Refactor-friendly**: Type errors surface during refactoring

## Domain Type Interface

```tsx
// frontend/src/types/activity.ts
export interface Activity {
  id: string;
  title: string;
  description: string;
  category: string;
  location: Location;
  createdAt: string;
}

export interface Location {
  id: string;
  name: string;
  slug: string;
  coordinates: {
    latitude: number;
    longitude: number;
  };
}
```

## Component Props Interface

```tsx
// frontend/src/components/activity/ActivityCard.tsx
interface ActivityCardProps {
  activity: Activity;
  onClick?: (activityId: string) => void;
  testId?: string;
}

export const ActivityCard = ({
  activity,
  onClick,
  testId
}: ActivityCardProps) => {
  return (
    <div data-testid={testId}>
      <h3>{activity.title}</h3>
      <p>{activity.description}</p>
    </div>
  );
};
```

## Common Patterns

### Props with Optional Fields

```tsx
interface CardProps {
  title: string;
  subtitle?: string;
  footer?: React.ReactNode;
  variant?: 'default' | 'highlighted';
}
```

### Props with Children

```tsx
interface LayoutProps {
  children: React.ReactNode;
  className?: string;
}

const Layout = ({ children, className }: LayoutProps) => {
  return <div className={className}>{children}</div>;
};
```

### Generic Types

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

### State Interface

```tsx
interface FormState {
  username: string;
  email: string;
  errors: {
    username?: string;
    email?: string;
  };
}

const Form = () => {
  const [state, setState] = useState<FormState>({
    username: '',
    email: '',
    errors: {}
  });
};
```

### API Response Interface

```tsx
interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

interface PaginatedResponse<T> extends ApiResponse<T[]> {
  total: number;
  page: number;
  pageSize: number;
}
```

## Guidelines

1. **Name with purpose**: Use descriptive names (Props, State, Response)
2. **Export when shared**: Export interfaces used across files
3. **Co-locate**: Define near usage when specific to one component
4. **Use optional**: Mark optional fields with `?`
5. **Avoid `any`**: Never use `any` type - defeats type safety

## Extending and Composing

### Union Types

```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'danger';
  size: 'small' | 'medium' | 'large';
}
```

### Extending Interfaces

```tsx
interface BaseProps {
  id: string;
  className?: string;
}

interface ButtonProps extends BaseProps {
  onClick: () => void;
  label: string;
}
```

### Intersection Types

```tsx
type WithLoading = { isLoading: boolean };
type WithError = { error: Error | null };
type DataState<T> = { data: T | null } & WithLoading & WithError;
```

## Common Mistakes

```tsx
// Bad: Using any
interface Props {
  data: any; // Defeats type safety!
}

// Good: Specific type
interface Props {
  data: Activity[];
}

// Bad: Inline types repeated
const Component1 = (props: { id: string; name: string }) => {};
const Component2 = (props: { id: string; name: string }) => {};

// Good: Shared interface
interface EntityProps {
  id: string;
  name: string;
}
const Component1 = (props: EntityProps) => {};
const Component2 = (props: EntityProps) => {};
```

## Testing with Interfaces

```tsx
const mockActivity: Activity = {
  id: '1',
  title: 'Test Activity',
  description: 'Test description',
  category: 'entertainment',
  location: {
    id: 'loc-1',
    name: 'Brighton',
    slug: 'brighton-uk',
    coordinates: { latitude: 50.8, longitude: -0.13 }
  },
  createdAt: '2024-01-01'
};
```

## Related Patterns

- Arrow Function Components: See `.ai/common-standards/typescript-components.md`

## References

- TypeScript Handbook: <https://www.typescriptlang.org/docs/handbook/interfaces.html>
- React TypeScript Cheatsheet: <https://react-typescript-cheatsheet.netlify.app/>
