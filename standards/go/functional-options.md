# Functional Options Pattern

Flexible constructor configuration using option functions that modify struct instances. Enables optional parameters without constructor explosion.

## Pattern

```go
type Option func(*Component)

func WithTimeout(timeout time.Duration) Option {
    return func(c *Component) {
        c.timeout = timeout
    }
}

func NewComponent(required string, opts ...Option) *Component {
    c := &Component{
        required: required,
        timeout:  30 * time.Second,  // default
    }
    for _, opt := range opts {
        opt(c)
    }
    return c
}
```

## Why Use This Pattern

- **Backward compatible**: Add options without breaking existing code
- **Self-documenting**: Option names describe their purpose
- **Type safe**: Compile-time checking of option types
- **Flexible**: Mix required and optional parameters cleanly
- **No constructor explosion**: Avoid NewComponentWithX variations

## Repository Options

```go
// pipeline/pkg/normaliser/repository.go:28
type RepositoryOption func(*normalisationRepository)

func WithLogger(logger *slog.Logger) RepositoryOption {
    return func(r *normalisationRepository) {
        r.logger = logger
    }
}

func NewRepository(pool *pgxpool.Pool, opts ...RepositoryOption) Repository {
    repo := &normalisationRepository{
        pool:   pool,
        logger: slog.Default(),  // default logger
    }
    for _, opt := range opts {
        opt(repo)
    }
    return repo
}
```

## Worker Options with Validation

```go
// pipeline/pkg/workers/normaliser/normaliser_worker.go:66
type WorkerOption func(*NormaliserWorker) error

func WithBatchSize(size int) WorkerOption {
    return func(w *NormaliserWorker) error {
        if size <= 0 {
            return fmt.Errorf("batch size must be positive")
        }
        w.config.BatchSize = size
        return nil
    }
}

func NewWorker(repo Repository, opts ...WorkerOption) (*NormaliserWorker, error) {
    w := &NormaliserWorker{
        repo: repo,
        config: WorkerConfig{
            BatchSize: 100,  // default
        },
    }
    for _, opt := range opts {
        if err := opt(w); err != nil {
            return nil, fmt.Errorf("failed to apply option: %w", err)
        }
    }
    return w, nil
}
```

## Usage Examples

```go
// Use defaults
worker := NewWorker(repo)

// Override specific options
worker := NewWorker(repo,
    WithBatchSize(50),
    WithLogger(logger),
)

// Mix required and optional parameters
client := NewClient(credentials,
    WithRateLimiter(limiter),
    WithTimeout(30*time.Second),
)
```

## Option Patterns

### Simple Option (No Error)

```go
func WithTimeout(timeout time.Duration) Option {
    return func(c *Component) {
        c.timeout = timeout
    }
}
```

### Validating Option (Returns Error)

```go
func WithBatchSize(size int) Option {
    return func(c *Component) error {
        if size <= 0 {
            return fmt.Errorf("batch size must be positive")
        }
        c.batchSize = size
        return nil
    }
}
```

### Complex Option (Multiple Fields)

```go
func WithDatabaseConfig(host string, port int) Option {
    return func(c *Component) {
        c.dbHost = host
        c.dbPort = port
    }
}
```

## Guidelines

1. **Name with prefix**: Use `With*` for option functions
2. **Return closure**: Option function returns modifier function
3. **Set defaults**: Establish sensible defaults before applying options
4. **Validate when needed**: Return errors for invalid option values
5. **Document behavior**: Clearly explain what each option does

## When to Use

**Use functional options when:**

- Component has multiple optional parameters
- Configuration is likely to expand over time
- Want backward compatibility when adding parameters
- Need runtime validation of optional values

**Use simple constructor when:**

- All parameters are required
- Configuration is stable and unlikely to change
- Only 1-2 optional parameters with clear defaults

## Common Mistakes

```go
// Bad: No defaults, required options
func NewComponent(opts ...Option) *Component {
    c := &Component{}  // Empty, requires options
    for _, opt := range opts {
        opt(c)
    }
    return c
}

// Good: Sensible defaults
func NewComponent(required string, opts ...Option) *Component {
    c := &Component{
        required: required,
        timeout:  30 * time.Second,
        retries:  3,
    }
    for _, opt := range opts {
        opt(c)
    }
    return c
}
```

## Related Patterns

- New* Constructors: See `.ai/common-standards/constructors.md`

## References

- Self-referential functions: <https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html>
- Functional options: <https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis>
