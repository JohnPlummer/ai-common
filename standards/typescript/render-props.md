# Render Props Pattern

Component pattern using function-as-child to share code and state between components with full TypeScript type safety.

## Pattern

```typescript
interface DataProviderProps<T> {
  fetchData: () => Promise<T>;
  children: (props: {
    data: T | null;
    loading: boolean;
    error: Error | null;
  }) => ReactNode;
}

export function DataProvider<T>({ fetchData, children }: DataProviderProps<T>) {
  const [state, setState] = useState<{
    data: T | null;
    loading: boolean;
    error: Error | null;
  }>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    fetchData()
      .then(data => setState({ data, loading: false, error: null }))
      .catch(error => setState({ data: null, loading: false, error }));
  }, [fetchData]);

  return <>{children(state)}</>;
}
```

## Why Use This Pattern

- **Flexible rendering**: Parent controls how data is rendered
- **Type safe**: Generic types provide full TypeScript inference
- **Reusable logic**: State management separated from presentation
- **Composable**: Easy to nest and combine render props
- **Clear contract**: Explicit props passed to render function

## Usage Example

```typescript
const ActivityListPage = () => {
  return (
    <DataProvider fetchData={() => ActivityService.getAll()}>
      {({ data, loading, error }) => {
        if (loading) return <Loading />;
        if (error) return <ErrorMessage error={error} />;
        if (!data) return <Empty />;

        return <ActivityList activities={data} />;
      }}
    </DataProvider>
  );
};
```

## Generic List Provider

```typescript
interface ListProviderProps<T> {
  items: T[];
  children: (props: {
    items: T[];
    selectedItem: T | null;
    selectItem: (item: T) => void;
    clearSelection: () => void;
  }) => ReactNode;
}

export function ListProvider<T>({ items, children }: ListProviderProps<T>) {
  const [selectedItem, setSelectedItem] = useState<T | null>(null);

  const selectItem = (item: T) => setSelectedItem(item);
  const clearSelection = () => setSelectedItem(null);

  return (
    <>
      {children({
        items,
        selectedItem,
        selectItem,
        clearSelection,
      })}
    </>
  );
}

// Usage
<ListProvider items={activities}>
  {({ items, selectedItem, selectItem }) => (
    <div>
      {items.map(item => (
        <ActivityCard
          key={item.id}
          activity={item}
          selected={selectedItem?.id === item.id}
          onClick={() => selectItem(item)}
        />
      ))}
    </div>
  )}
</ListProvider>
```

## Mouse Position Tracker

```typescript
interface MouseProviderProps {
  children: (props: { x: number; y: number }) => ReactNode;
}

const MouseProvider = ({ children }: MouseProviderProps) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return <>{children(position)}</>;
};

// Usage
<MouseProvider>
  {({ x, y }) => (
    <div>
      Mouse position: {x}, {y}
    </div>
  )}
</MouseProvider>
```

## Render Props with Component Prop

Alternative pattern using render prop:

```typescript
interface DataLoaderProps<T> {
  fetchData: () => Promise<T>;
  render: (props: {
    data: T | null;
    loading: boolean;
    error: Error | null;
  }) => ReactNode;
}

export function DataLoader<T>({ fetchData, render }: DataLoaderProps<T>) {
  // Same implementation as DataProvider
  return <>{render(state)}</>;
}

// Usage
<DataLoader
  fetchData={() => ActivityService.getAll()}
  render={({ data, loading, error }) => {
    if (loading) return <Loading />;
    return <ActivityList activities={data} />;
  }}
/>
```

## Guidelines

1. **Use generics**: Make components type-safe with generic parameters
2. **Clear props**: Explicitly type the props passed to children function
3. **Meaningful names**: Name render prop parameters descriptively
4. **Handle all states**: Pass loading, error, and data states
5. **Consider hooks**: For simple cases, custom hooks may be cleaner

## Render Props vs Custom Hooks

**Use Render Props when:**

- Need to render something based on component state
- Multiple components need same logic but different rendering
- Want explicit control over rendering

**Use Custom Hooks when:**

- Only need data/logic, not rendering
- Cleaner API for simple data fetching
- Want to compose multiple hooks together

## Testing Render Props

```typescript
describe('DataProvider', () => {
  it('should pass loading state initially', () => {
    let renderProps: any;

    render(
      <DataProvider fetchData={() => Promise.resolve([])}>
        {props => {
          renderProps = props;
          return null;
        }}
      </DataProvider>
    );

    expect(renderProps.loading).toBe(true);
    expect(renderProps.data).toBeNull();
  });

  it('should pass data after fetch', async () => {
    const mockData = [{ id: '1', name: 'Test' }];

    const { rerender } = render(
      <DataProvider fetchData={() => Promise.resolve(mockData)}>
        {({ data, loading }) => (
          <div>
            {loading ? 'Loading...' : data?.length}
          </div>
        )}
      </DataProvider>
    );

    await waitFor(() => {
      expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
    });

    expect(screen.getByText('1')).toBeInTheDocument();
  });
});
```

## Related Patterns

- Custom Hooks: See `.ai/common-standards/custom-hooks.md`
- TypeScript Components: See `.ai/common-standards/typescript-components.md`

## References

- Render Props: <https://react.dev/reference/react/cloneElement#passing-data-with-a-render-prop>
- TypeScript Generics: <https://www.typescriptlang.org/docs/handbook/2/generics.html>
