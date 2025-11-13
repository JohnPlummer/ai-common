# Custom Hooks Pattern

Reusable React hooks for data fetching, state management, and side effects with loading and error states.

## Pattern

```typescript
export const useActivityDetail = (id: string) => {
  const [activity, setActivity] = useState<Activity | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchActivity = async () => {
      try {
        setLoading(true);
        const data = await ActivityService.getActivity(id);
        setActivity(data);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchActivity();
  }, [id]);

  return { activity, loading, error };
};
```

## Why Use This Pattern

- **Reusable**: Extract common logic shared across components
- **Composable**: Combine multiple hooks for complex behavior
- **Testable**: Unit test hooks independently of components
- **Type safe**: Full TypeScript inference for state and returns
- **Separation of concerns**: Logic isolated from presentation

## Data Fetching Hook

```typescript
// hooks/useActivityDetail.ts
export const useActivityDetail = (id: string) => {
  const [activity, setActivity] = useState<Activity | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchActivity = async () => {
      try {
        setLoading(true);
        const data = await ActivityService.getActivity(id);
        setActivity(data);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchActivity();
  }, [id]);

  return { activity, loading, error };
};
```

## Usage in Component

```typescript
const ActivityDetailPage = () => {
  const { id } = useParams();
  const { activity, loading, error } = useActivityDetail(id!);

  if (loading) return <Loading />;
  if (error) return <ErrorMessage error={error} />;
  if (!activity) return <NotFound />;

  return <ActivityCard activity={activity} />;
};
```

## Generic Data Fetching Hook

```typescript
export function useData<T>(
  fetchFn: () => Promise<T>,
  dependencies: any[] = []
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      try {
        setLoading(true);
        const result = await fetchFn();
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return { data, loading, error };
}

// Usage
const { data: activities } = useData(() => ActivityService.getAll());
```

## Form Hook

```typescript
export function useForm<T>(initialValues: T) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

  const handleChange = (field: keyof T) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues(prev => ({
      ...prev,
      [field]: e.target.value,
    }));
    // Clear error on change
    setErrors(prev => ({ ...prev, [field]: undefined }));
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };

  return { values, errors, handleChange, setErrors, reset };
}
```

## Debounce Hook

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage: Debounced search
const SearchBar = () => {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 500);

  useEffect(() => {
    if (debouncedSearch) {
      performSearch(debouncedSearch);
    }
  }, [debouncedSearch]);

  return <input value={search} onChange={e => setSearch(e.target.value)} />;
};
```

## Guidelines

1. **Name with use prefix**: All hooks must start with `use`
2. **Return objects**: Use `{ data, loading, error }` pattern for consistency
3. **Type returns**: Explicitly type return values for better inference
4. **Handle cleanup**: Return cleanup function from useEffect
5. **Cancel requests**: Prevent state updates after unmount

## Testing Custom Hooks

```typescript
import { renderHook, waitFor } from '@testing-library/react';

describe('useActivityDetail', () => {
  it('should fetch activity data', async () => {
    const mockActivity = { id: '1', name: 'Test Activity' };
    jest.spyOn(ActivityService, 'getActivity').mockResolvedValue(mockActivity);

    const { result } = renderHook(() => useActivityDetail('1'));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.activity).toEqual(mockActivity);
    expect(result.current.error).toBeNull();
  });
});
```

## Related Patterns

- React Hooks: See `.ai/common-standards/react-hooks.md`
- TypeScript Components: See `.ai/common-standards/typescript-components.md`

## References

- React Hooks Documentation: <https://react.dev/reference/react>
- Custom Hooks Guide: <https://react.dev/learn/reusing-logic-with-custom-hooks>
