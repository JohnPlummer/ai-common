# React Arrow Function Components

Functional React components using arrow function syntax with TypeScript. Modern, concise pattern for component definition.

## Pattern

```tsx
const ComponentName = () => {
  const [state, setState] = useState<Type>(initialValue);

  return (
    <div>
      {/* JSX content */}
    </div>
  );
};

export default ComponentName;
```

## Why Use This Pattern

- **Concise**: Less boilerplate than function declarations
- **Consistent**: Uniform component style across codebase
- **Hooks compatible**: Works seamlessly with React hooks
- **Modern**: Current React community standard
- **Type safe**: Full TypeScript support for props and state

## Page Component

```tsx
// frontend/src/pages/ActivityListPage.tsx:26
const ActivityListPage = () => {
  const [activities, setActivities] = useState<Activity[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const loadActivities = async () => {
      const data = await api.getActivities();
      setActivities(data);
      setIsLoading(false);
    };
    loadActivities();
  }, []);

  return (
    <div>
      {isLoading ? <Loading /> : <ActivityList activities={activities} />}
    </div>
  );
};
```

## Component with Props

```tsx
// frontend/src/components/MainContent.tsx:25
interface MainContentProps {
  children: React.ReactNode;
}

const MainContent = ({ children }: MainContentProps) => {
  return (
    <Box component="main" sx={{ flexGrow: 1, p: 3 }}>
      {children}
    </Box>
  );
};
```

## Component Patterns

### Simple Component (No Props)

```tsx
const SimpleComponent = () => {
  return <div>Content</div>;
};
```

### Component with Props Interface

```tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

const Button = ({ label, onClick, disabled = false }: ButtonProps) => {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
};
```

### Component with State

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

### Component with Effects

```tsx
const DataLoader = () => {
  const [data, setData] = useState<Data | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      const result = await api.getData();
      setData(result);
    };
    fetchData();
  }, []);

  if (!data) return <Loading />;
  return <DataView data={data} />;
};
```

## Guidelines

1. **Use arrow functions**: Consistent with codebase style
2. **Define interfaces**: Separate interface for props above component
3. **Destructure props**: Extract props in function parameter
4. **Type all state**: Use generics with useState for type safety
5. **Export default**: Single component per file with default export

## Common Patterns

### Props with Defaults

```tsx
interface Props {
  title: string;
  showIcon?: boolean;
  variant?: 'primary' | 'secondary';
}

const Card = ({
  title,
  showIcon = true,
  variant = 'primary'
}: Props) => {
  return (
    <div className={variant}>
      {showIcon && <Icon />}
      <h2>{title}</h2>
    </div>
  );
};
```

### Event Handlers

```tsx
const Form = () => {
  const [value, setValue] = useState('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Submit logic
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={value} onChange={handleChange} />
      <button type="submit">Submit</button>
    </form>
  );
};
```

## Testing

```tsx
describe('ActivityCard', () => {
  it('renders activity details', () => {
    const activity = {
      id: '1',
      title: 'Test Activity',
      category: 'entertainment'
    };

    render(<ActivityCard activity={activity} />);

    expect(screen.getByText('Test Activity')).toBeInTheDocument();
    expect(screen.getByText('entertainment')).toBeInTheDocument();
  });
});
```

## Related Patterns

- useState Hook: See `.ai/common-standards/react-hooks.md`
- TypeScript Interfaces: See `.ai/common-standards/typescript-interfaces.md`

## References

- React documentation: <https://react.dev/learn/your-first-component>
- TypeScript + React: <https://react-typescript-cheatsheet.netlify.app/>
