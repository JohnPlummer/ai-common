# React Hooks Pattern

React hooks for state management and side effects in functional components. Uses useState for local state and useEffect for lifecycle operations.

## useState Pattern

```tsx
const [state, setState] = useState<Type>(initialValue);
```

## useEffect Pattern

```tsx
useEffect(() => {
  // Side effect code
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

## Why Use These Patterns

- **Functional components**: Hooks work with function components
- **Type safe**: Full TypeScript generic support
- **Composable**: Combine multiple hooks in one component
- **Lifecycle control**: useEffect handles mount, update, unmount

## useState for Component State

```tsx
// frontend/src/pages/ActivityListPage.tsx:27
const ActivityListPage = () => {
  const [activities, setActivities] = useState<Activity[]>([]);
  const [categories, setCategories] = useState<string[]>([]);
  const [selectedCategory, setSelectedCategory] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);
};
```

## useEffect for Data Fetching

```tsx
// frontend/src/pages/ActivityListPage.tsx:44
useEffect(() => {
  const fetchCategories = async () => {
    try {
      const cats = await api.getCategories();
      setCategories(cats);
    } catch (error) {
      console.error('Failed to load categories:', error);
    }
  };
  fetchCategories();
}, []); // Empty array = run once on mount
```

## useEffect with Dependencies

```tsx
// frontend/src/pages/ActivityListPage.tsx:37
useEffect(() => {
  const searchParams = new URLSearchParams(location.search);
  const categoryParam = searchParams.get('category');
  setSelectedCategory(categoryParam);
}, [location.search]); // Re-run when URL changes
```

## Common Patterns

### Multiple State Variables

```tsx
const Component = () => {
  const [data, setData] = useState<Data[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
};
```

### State with Objects

```tsx
interface FormState {
  username: string;
  email: string;
}

const Form = () => {
  const [form, setForm] = useState<FormState>({
    username: '',
    email: ''
  });

  const updateField = (field: keyof FormState, value: string) => {
    setForm(prev => ({ ...prev, [field]: value }));
  };
};
```

### Effect with Cleanup

```tsx
useEffect(() => {
  const subscription = api.subscribe(data => setData(data));

  return () => {
    subscription.unsubscribe();
  };
}, []);
```

### Conditional Effects

```tsx
useEffect(() => {
  if (!userId) return;

  const fetchUserData = async () => {
    const data = await api.getUser(userId);
    setUser(data);
  };
  fetchUserData();
}, [userId]);
```

## Guidelines

1. **Type state explicitly**: Use generics with useState
2. **One concern per state**: Separate state for independent data
3. **List dependencies**: Include all values used in effect
4. **Clean up effects**: Return cleanup function when needed
5. **Guard effects**: Add early returns for conditional logic

## Common Mistakes

```tsx
// Bad: Mutating state directly
const [items, setItems] = useState([]);
items.push(newItem); // Don't mutate!

// Good: Create new array
setItems([...items, newItem]);

// Bad: Missing dependencies
useEffect(() => {
  if (condition) {
    doSomething(value);
  }
}, []); // Missing condition, value

// Good: All dependencies listed
useEffect(() => {
  if (condition) {
    doSomething(value);
  }
}, [condition, value]);

// Bad: Effect runs every render
useEffect(() => {
  fetchData();
}); // No dependency array!

// Good: Controlled execution
useEffect(() => {
  fetchData();
}, [someValue]);
```

## Testing

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('updates state on button click', async () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: 'Increment' });
  await userEvent.click(button);

  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

## Related Patterns

- Arrow Function Components: See `.ai/common-standards/typescript-components.md`

## References

- React hooks: <https://react.dev/reference/react>
- useState: <https://react.dev/reference/react/useState>
- useEffect: <https://react.dev/reference/react/useEffect>
