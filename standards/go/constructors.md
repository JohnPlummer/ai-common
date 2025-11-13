# New* Constructor Pattern

Standard Go constructor pattern using `New*` prefix for creating and initializing instances with explicit dependencies.

## Pattern

```go
func NewService(repo Repository, logger *slog.Logger) *Service {
    return &Service{
        repo:   repo,
        logger: logger,
    }
}
```

## Why Use This Pattern

- **Explicit dependencies**: All dependencies visible in function signature
- **Type safety**: Compile-time checking of required dependencies
- **Testability**: Easy to inject mocks during testing
- **Convention**: Standard Go naming pattern recognized ecosystem-wide

## Simple Constructor

```go
// backend/pkg/api/services/location_service.go:67
func NewLocationService(repo LocationRepository) LocationService {
    return &locationService{
        repository: repo,
    }
}
```

## Constructor with Validation

```go
// pipeline/pkg/workers/deduplicator/processor.go:142
func NewProcessor(
    repo DeduplicationRepository,
    llm LLMClient,
    logger *slog.Logger,
) (*Processor, error) {
    if repo == nil {
        return nil, fmt.Errorf("repository cannot be nil")
    }
    if llm == nil {
        return nil, fmt.Errorf("LLM client cannot be nil")
    }

    return &Processor{
        repo:   repo,
        llm:    llm,
        logger: logger,
    }, nil
}
```

## Constructor Patterns

### Basic Constructor (No Error)

Use when initialization cannot fail:

```go
func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}
```

### Constructor with Error Return

Use when validation or setup can fail:

```go
func NewService(repo Repository) (*Service, error) {
    if repo == nil {
        return nil, fmt.Errorf("repository required")
    }
    return &Service{repo: repo}, nil
}
```

### Constructor with Default Values

```go
func NewService(repo Repository, logger *slog.Logger) *Service {
    if logger == nil {
        logger = slog.Default()
    }
    return &Service{
        repo:   repo,
        logger: logger,
    }
}
```

## Guidelines

1. **Name consistently**: Always prefix with `New`
2. **Return interface**: When applicable, return interface type not concrete
3. **Validate dependencies**: Check for nil required dependencies
4. **Return errors**: Use error return when validation or setup can fail
5. **Initialize fields**: Set all required fields in constructor

## Common Mistakes

```go
// Bad: No constructor, direct struct initialization
service := &Service{repo: repo}

// Good: Use constructor for initialization
service := NewService(repo)

// Bad: Constructor doesn't validate
func NewService(repo Repository) *Service {
    return &Service{repo: repo}  // repo could be nil!
}

// Good: Validate required dependencies
func NewService(repo Repository) (*Service, error) {
    if repo == nil {
        return nil, fmt.Errorf("repository required")
    }
    return &Service{repo: repo}, nil
}
```

## Testing with Constructors

```go
// Unit test with mock
mockRepo := mocks.NewMockRepository(t)
service := NewService(mockRepo)

// Integration test with real implementation
repo := postgres.NewRepository(db)
service := NewService(repo)
```

## Related Patterns

- Functional Options: See `.ai/common-standards/functional-options.md`

## References

- Effective Go: <https://go.dev/doc/effective_go#composite_literals>
- Go Code Review Comments: <https://github.com/golang/go/wiki/CodeReviewComments>
