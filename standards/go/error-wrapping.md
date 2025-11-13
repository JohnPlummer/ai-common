# Error Wrapping with fmt.Errorf

Standard Go 1.13+ error wrapping using the `%w` verb to create error chains that preserve the original error while adding context.

## Pattern

```go
if err != nil {
    return fmt.Errorf("operation description: %w", err)
}
```

## Why Use This Pattern

- **Error chains**: Preserves original error for `errors.Is()` and `errors.As()` checking
- **Context**: Adds meaningful context at each layer
- **Debugging**: Stack of context helps trace error origin
- **Standard**: Official Go 1.13+ pattern, widely adopted

## Examples from Codebase

### Repository Error Wrapping

```go
// backend/pkg/api/repositories/postgres_activity_repository.go:54
if err != nil {
    return nil, fmt.Errorf("failed to query activities: %w", err)
}
```

### Worker Error Wrapping

```go
// pipeline/pkg/workers/reddit_worker.go:196
if err != nil {
    return fmt.Errorf("failed to fetch posts from r/%s: %w", w.subreddit, err)
}
```

### Multi-layer Context

```go
// Service layer
func (s *Service) ProcessItem(ctx context.Context, id string) error {
    item, err := s.repo.GetItem(ctx, id)
    if err != nil {
        return fmt.Errorf("failed to get item %s: %w", id, err)
    }
    // ... process item
}

// Repository layer
func (r *Repository) GetItem(ctx context.Context, id string) (*Item, error) {
    var item Item
    err := r.db.QueryRow(ctx, query, id).Scan(&item)
    if err != nil {
        return nil, fmt.Errorf("failed to query item: %w", err)
    }
    return &item, nil
}
```

## Error Checking with Wrapped Errors

```go
err := service.ProcessItem(ctx, "123")
if err != nil {
    // Check for specific error types
    if errors.Is(err, pgx.ErrNoRows) {
        // Handle not found
    }

    // Unwrap to check underlying error
    var netErr net.Error
    if errors.As(err, &netErr) && netErr.Timeout() {
        // Handle timeout
    }
}
```

## Guidelines

1. **Always use %w** for error wrapping (not %v)
2. **Add context** describing what operation failed
3. **Include identifiers** (IDs, names) when relevant
4. **Don't repeat error text** - let wrapping build context stack
5. **Use descriptive operation names** - helps debugging

## Anti-patterns

```go
// Bad: Uses %v instead of %w (breaks error chain)
return fmt.Errorf("failed to process: %v", err)

// Bad: Too vague, no useful context
return fmt.Errorf("error: %w", err)

// Bad: Repeats information already in error
return fmt.Errorf("database error: failed to query: %w", err)

// Good: Clear context, uses %w
return fmt.Errorf("failed to query activities for location %s: %w", locationID, err)
```

## Related Patterns

- jp-go-errors.md: Standardized error types and sentinel errors
- Error Classification: Use jp-go-errors for retry detection (IsRetryable pattern)
- Custom Error Types: Embed errors with additional fields

## References

- Go 1.13 error wrapping: <https://go.dev/blog/go1.13-errors>
- Standard library: `errors.Is()`, `errors.As()`, `errors.Unwrap()`
