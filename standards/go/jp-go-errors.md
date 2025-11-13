# jp-go-errors Package Usage

*Category: go*

## When to Use

Use jp-go-errors when you need standardized error handling across Go projects with:

- Sentinel errors for common conditions (not found, validation, etc.)
- Error constructors with consistent formatting
- Error wrapping with context preservation
- HTTP status code integration
- Retry detection for transient failures

## Package Overview

jp-go-errors provides standardized error types and sentinels that maintain semantic meaning across service boundaries.

Repository: `github.com/JohnPlummer/jp-go-errors`

## Import Convention

```go
import (
    pkgerrors "github.com/JohnPlummer/jp-go-errors"
)
```

Use the `pkgerrors` alias to avoid conflict with standard library `errors` package.

## Common Sentinel Errors

Sentinel errors represent specific, well-known error conditions:

```go
// Check for sentinel errors
if errors.Is(err, pkgerrors.ErrNotFound) {
    // Handle not found case
}

if errors.Is(err, pkgerrors.ErrValidation) {
    // Handle validation failure
}

if errors.Is(err, pkgerrors.ErrRateLimited) {
    // Handle rate limiting
}
```

### Available Sentinels

- `ErrNotFound` - Resource not found
- `ErrValidation` - Input validation failed
- `ErrRateLimited` - Rate limit exceeded
- `ErrTimeout` - Operation timed out
- `ErrConflict` - Resource conflict (e.g., duplicate)
- `ErrUnauthorized` - Authentication required
- `ErrForbidden` - Permission denied

## Error Constructors

Create errors with consistent formatting:

```go
// Create a not found error
err := pkgerrors.NewNotFoundError("user", "id", userID)
// Returns: "user not found: id=123"

// Create a validation error
err := pkgerrors.NewValidationError("email must be valid format")

// Create an internal error
err := pkgerrors.NewInternalError("database connection failed")
```

### Constructor Patterns

```go
// NotFoundError with entity and identifier
pkgerrors.NewNotFoundError(entity, field, value string) error

// ValidationError with message
pkgerrors.NewValidationError(message string) error

// InternalError with message
pkgerrors.NewInternalError(message string) error

// NetworkError with message
pkgerrors.NewNetworkError(message string) error
```

## Error Wrapping

Preserve error chains while adding context:

```go
result, err := repository.GetActivity(ctx, id)
if err != nil {
    return nil, fmt.Errorf("failed to get activity: %w", err)
}
```

jp-go-errors types preserve their identity through wrapping, so `errors.Is()` works correctly:

```go
err := someFunc()
if errors.Is(err, pkgerrors.ErrNotFound) {
    // This works even if err was wrapped multiple times
}
```

## HTTP Status Code Integration

jp-go-errors types implement HTTPError interface for web handlers:

```go
type HTTPError interface {
    StatusCode() int
}

// Check if error has HTTP status
if httpErr, ok := err.(pkgerrors.HTTPError); ok {
    statusCode := httpErr.StatusCode()
    // Use statusCode in HTTP response
}
```

### Default Status Codes

- NotFoundError → 404
- ValidationError → 400
- UnauthorizedError → 401
- ForbiddenError → 403
- ConflictError → 409
- RateLimitError → 429
- InternalError → 500
- NetworkError → 503

## Retry Detection

Check if an error represents a transient condition:

```go
if pkgerrors.IsRetryable(err) {
    // Retry the operation
}
```

IsRetryable returns true for:

- Any error implementing `Retryable` interface
- NetworkError
- RateLimitError
- Timeout errors
- Context deadline exceeded

## Helper Functions

```go
// Check if error or its cause is NotFound
if pkgerrors.IsNotFound(err) {
    return 404, "resource not found"
}

// Works with standard errors wrapped as NotFoundError
err := fmt.Errorf("user lookup failed: %w", pkgerrors.ErrNotFound)
pkgerrors.IsNotFound(err) // true
```

## Usage in Repositories

```go
func (r *Repository) GetUser(ctx context.Context, id string) (*User, error) {
    var user User
    err := r.db.QueryRow(ctx, "SELECT * FROM users WHERE id = $1", id).Scan(&user)

    if errors.Is(err, pgx.ErrNoRows) {
        return nil, pkgerrors.NewNotFoundError("user", "id", id)
    }

    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }

    return &user, nil
}
```

## Usage in HTTP Handlers

```go
func (h *Handler) GetUser(w http.ResponseWriter, r *http.Request) {
    user, err := h.service.GetUser(r.Context(), id)

    if err != nil {
        if pkgerrors.IsNotFound(err) {
            http.Error(w, "User not found", http.StatusNotFound)
            return
        }

        // Check for HTTP status code
        if httpErr, ok := err.(pkgerrors.HTTPError); ok {
            http.Error(w, err.Error(), httpErr.StatusCode())
            return
        }

        http.Error(w, "Internal error", http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

## Testing with Sentinels

```go
It("returns ErrNotFound for missing user", func() {
    _, err := repo.GetUser(ctx, "missing-id")

    Expect(errors.Is(err, pkgerrors.ErrNotFound)).To(BeTrue())
    Expect(pkgerrors.IsNotFound(err)).To(BeTrue())
})

It("returns validation error for invalid email", func() {
    err := service.CreateUser(ctx, &User{Email: "invalid"})

    Expect(errors.Is(err, pkgerrors.ErrValidation)).To(BeTrue())
})
```

## Related Standards

- error-wrapping.md - Generic Go error wrapping patterns
- http-middleware.md - HTTP error handling middleware

## Common Pitfalls

1. **Don't create new sentinel instances** - Always use the package sentinels:

   ```go
   // Wrong
   return fmt.Errorf("not found")

   // Right
   return pkgerrors.ErrNotFound
   return pkgerrors.NewNotFoundError("user", "id", id)
   ```

2. **Use errors.Is() not ==** - Wrapped errors need errors.Is():

   ```go
   // Wrong
   if err == pkgerrors.ErrNotFound

   // Right
   if errors.Is(err, pkgerrors.ErrNotFound)
   ```

3. **Preserve error chains** - Always wrap with %w:

   ```go
   // Wrong
   return fmt.Errorf("operation failed: %v", err)

   // Right
   return fmt.Errorf("operation failed: %w", err)
   ```
