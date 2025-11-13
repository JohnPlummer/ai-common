# Retry Logic Pattern

Exponential backoff with jitter using sethvargo/go-retry for transient failures.

## Pattern

```go
import "github.com/sethvargo/go-retry"

// Standard retry configuration
b := retry.NewExponential(1 * time.Second)
b = retry.WithMaxRetries(3, b)
b = retry.WithJitter(100*time.Millisecond, b)

err := retry.Do(ctx, b, func(ctx context.Context) error {
    result, err := externalService.Call()
    if err != nil {
        // Only retry transient errors
        if errors.Is(err, ErrRateLimited) || errors.Is(err, ErrTimeout) {
            return retry.RetryableError(err)
        }
        // Don't retry permanent errors
        return err
    }
    return nil
})
```

## Why Use This Pattern

- **Handles transient failures**: Network blips, rate limits, temporary outages
- **Exponential backoff**: Increasingly longer delays prevent overwhelming failing service
- **Jitter**: Prevents thundering herd when multiple clients retry simultaneously
- **Context aware**: Respects cancellation and timeouts
- **Standard library**: Seth Vargo's retry is widely used

## Configuration

Standard retry parameters:

```go
type RetryConfig struct {
    MaxRetries     int           // Default: 3
    InitialBackoff time.Duration // Default: 1s
    MaxBackoff     time.Duration // Default: 30s
    JitterEnabled  bool          // Default: true
    Jitter         time.Duration // Default: 100ms
}
```

Backoff progression: 1s → 2s → 4s (doubles each retry)

Adjust based on:

- **Service SLA**: Higher retries for services with frequent transients
- **User experience**: Lower retries for user-facing operations
- **Cost**: Expensive operations may warrant fewer retries

## When to Retry

**Retry these errors**:

- Network timeouts
- Rate limiting (429)
- Service temporarily unavailable (503)
- Database deadlocks

**Don't retry these**:

- Validation errors (400)
- Authentication errors (401, 403)
- Not found (404)
- Internal logic errors

Mark retryable errors explicitly:

```go
if isTransient(err) {
    return retry.RetryableError(err)
}
return err // Permanent failure
```

## Combining with Circuit Breaker

Use both patterns together for maximum resilience:

```go
// Circuit breaker wraps retry logic
result, err := circuitBreaker.Execute(func() (any, error) {
    var result any
    err := retry.Do(ctx, backoff, func(ctx context.Context) error {
        result, err = service.Call()
        if isTransient(err) {
            return retry.RetryableError(err)
        }
        return err
    })
    return result, err
})
```

Circuit breaker prevents retrying when service is known to be down.

## Example from Codebase

**Location**: `pipeline/pkg/llm/retry.go`

Retry logic for OpenAI API calls with exponential backoff and rate limit handling.

## Related Patterns

- Circuit breaker: `.ai/project-standards/circuit-breaker.md`
- Error classification: `.ai/project-standards/error-classification.md`
