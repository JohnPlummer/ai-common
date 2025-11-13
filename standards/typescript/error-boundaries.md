# Error Boundary Pattern

React class component for catching JavaScript errors anywhere in component tree, logging errors, and displaying fallback UI.

## Pattern

```typescript
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, ErrorBoundaryState> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <FallbackUI error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

## Why Use This Pattern

- **Graceful degradation**: Prevents entire app from crashing
- **Error isolation**: Contains errors to specific subtrees
- **User experience**: Shows friendly error message instead of blank screen
- **Logging**: Centralized place to log or report errors
- **Recovery**: Provides reset mechanism for users

## Complete Error Boundary

```typescript
// components/ErrorBoundary.tsx
interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);

    // Call custom error handler if provided
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <Box p={4}>
          <Typography variant="h5">Something went wrong</Typography>
          <Typography variant="body1" color="text.secondary">
            {this.state.error?.message}
          </Typography>
          <Button
            variant="contained"
            onClick={() => window.location.reload()}
            sx={{ mt: 2 }}
          >
            Reload Page
          </Button>
        </Box>
      );
    }

    return this.props.children;
  }
}
```

## Usage in App

```typescript
// App.tsx
const App = () => {
  return (
    <ErrorBoundary
      fallback={<ErrorFallback />}
      onError={(error, errorInfo) => {
        // Log to error reporting service
        logErrorToService(error, errorInfo);
      }}
    >
      <Router>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/activities/:id" element={<ActivityDetailPage />} />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
};
```

## Nested Error Boundaries

```typescript
const ActivitySection = () => {
  return (
    <ErrorBoundary fallback={<ActivityErrorFallback />}>
      <ActivityList />
    </ErrorBoundary>
  );
};

const App = () => {
  return (
    <ErrorBoundary fallback={<AppErrorFallback />}>
      <Header />
      <ActivitySection /> {/* Isolated error boundary */}
      <Footer />
    </ErrorBoundary>
  );
};
```

## Custom Fallback UI

```typescript
interface ErrorFallbackProps {
  error?: Error | null;
  resetError?: () => void;
}

const ErrorFallback = ({ error, resetError }: ErrorFallbackProps) => {
  return (
    <Box
      display="flex"
      flexDirection="column"
      alignItems="center"
      justifyContent="center"
      minHeight="400px"
      p={3}
    >
      <ErrorOutlineIcon color="error" sx={{ fontSize: 60, mb: 2 }} />
      <Typography variant="h4" gutterBottom>
        Oops! Something went wrong
      </Typography>
      <Typography variant="body1" color="text.secondary" mb={3}>
        {error?.message || 'An unexpected error occurred'}
      </Typography>
      <Button variant="contained" onClick={resetError || (() => window.location.reload())}>
        Try Again
      </Button>
    </Box>
  );
};
```

## Guidelines

1. **Use class components**: Error boundaries must be class components (no hooks equivalent)
2. **Implement both methods**: Use `getDerivedStateFromError` for state and `componentDidCatch` for logging
3. **Provide fallback**: Always show user-friendly error message
4. **Enable recovery**: Provide button to retry or reload
5. **Log errors**: Use `componentDidCatch` to send errors to logging service

## What Error Boundaries Catch

**Catches:**

- Errors during rendering
- Errors in lifecycle methods
- Errors in constructors of child components

**Does NOT catch:**

- Event handlers (use try/catch)
- Asynchronous code (setTimeout, promises)
- Server-side rendering
- Errors in error boundary itself

## Testing Error Boundaries

```typescript
describe('ErrorBoundary', () => {
  it('should render children when no error', () => {
    render(
      <ErrorBoundary>
        <div>Child content</div>
      </ErrorBoundary>
    );

    expect(screen.getByText('Child content')).toBeInTheDocument();
  });

  it('should render fallback on error', () => {
    const ThrowError = () => {
      throw new Error('Test error');
    };

    render(
      <ErrorBoundary>
        <ThrowError />
      </ErrorBoundary>
    );

    expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
  });
});
```

## Related Patterns

- Custom Hooks: See `.ai/common-standards/custom-hooks.md`
- TypeScript Components: See `.ai/common-standards/typescript-components.md`

## References

- Error Boundaries Documentation: <https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary>
