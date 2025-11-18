# jp-go-resilience Package Usage

*Category: go*

Standardized circuit breakers, retry logic, and health checking with jp-go-errors integration.

## When to Use

Use jp-go-resilience for fault-tolerant clients:

- External API calls (HTTP, gRPC) needing circuit breaker protection
- Transient failure handling with exponential backoff retry
- Health status monitoring for downstream services
- Combining retry + circuit breaker for layered resilience

## Package Overview

jp-go-resilience wraps sony/gobreaker and sethvargo/go-retry with:

- Pre-configured circuit breaker settings (configurable thresholds, timeouts)
- Exponential/constant/fibonacci backoff strategies with jitter
- Integration with jp-go-errors.IsRetryable() for smart retry decisions
- Generic interface supporting any request/response types
- Health status reporting for monitoring

## Import Convention

```go
import "github.com/JohnPlummer/jp-go-resilience"
```

## Circuit Breaker Usage

Wraps clients to prevent cascading failures:

```go
// Implement ResilientClient interface
type HTTPClient struct {
    client *http.Client
}

func (c *HTTPClient) Execute(ctx context.Context, req *http.Request) (*http.Response, error) {
    return c.client.Do(req.WithContext(ctx))
}

// Wrap with circuit breaker
cbClient := resilience.NewCircuitBreakerWrapper(
    httpClient,
    resilience.WithMaxRequests(5),      // Half-open state allows 5 requests
    resilience.WithTimeout(60*time.Second), // Open → half-open after 60s
)

resp, err := cbClient.Execute(ctx, req)
```

**Circuit breaker states:**

- **Closed**: Normal operation, tracks failures
- **Open**: Rejects requests immediately (fast fail)
- **Half-Open**: Allows limited requests to test recovery

**Configuration options:**

- `WithMaxRequests(n)` - Concurrent requests in half-open state
- `WithTimeout(d)` - Duration before transitioning open → half-open
- `WithInterval(d)` - Window for tracking failure rate
- `WithErrorClassifier(c)` - Custom error classification

## Retry Logic Usage

Handles transient failures with backoff:

```go
// Exponential backoff (default 2.0 multiplier)
retryClient := resilience.NewRetryWrapper(
    httpClient,
    resilience.WithMaxAttempts(5),
    resilience.WithExponentialBackoff(time.Second, 30*time.Second),
)

// Custom multiplier for moderate growth
retryClient := resilience.NewRetryWrapper(
    httpClient,
    resilience.WithMaxAttempts(5),
    resilience.WithExponentialBackoff(time.Second, 30*time.Second),
    resilience.WithMultiplier(1.5), // 50% growth vs 100% doubling
)

// Constant backoff with jitter
retryClient := resilience.NewRetryWrapper(
    httpClient,
    resilience.WithMaxAttempts(3),
    resilience.WithConstantBackoff(2*time.Second),
)
```

**Backoff strategies:**

- **Exponential**: Delay doubles each retry (configurable multiplier)
- **Constant**: Fixed delay between retries (with jitter)
- **Fibonacci**: Fibonacci sequence delays

## Combining Retry + Circuit Breaker

Layered resilience (circuit breaker inner, retry outer):

```go
combined := resilience.CombineRetryAndCircuitBreaker(
    httpClient,
    &resilience.RetryConfig{
        MaxAttempts:  3,
        InitialDelay: time.Second,
        MaxDelay:     30 * time.Second,
        Strategy:     resilience.RetryStrategyExponential,
    },
    &resilience.CircuitBreakerConfig{
        MaxRequests: 5,
        Timeout:     60 * time.Second,
    },
    logger,
)
```

**Ordering matters:** Circuit breaker prevents retries to failing services.

## Integration with jp-go-errors

Automatic retry detection:

```go
import (
    "github.com/JohnPlummer/jp-go-errors"
    "github.com/JohnPlummer/jp-go-resilience"
)

// Default classifier uses jp-go-errors.IsRetryable()
retryClient := resilience.NewRetryWrapper(
    client,
    resilience.WithMaxAttempts(3),
)

// HTTPError with 429, 500-504 → retries automatically
// ValidationError, context.Canceled → fails fast
```

**Custom classification:**

```go
classifier := func(err error) bool {
    // Retry on network timeouts only
    return errors.Is(err, jperrors.ErrNetworkTimeout)
}

retryClient := resilience.NewRetryWrapper(
    client,
    resilience.WithErrorClassifier(resilience.ErrorClassifierFunc(classifier)),
)
```

## Health Status

Monitor circuit breaker state:

```go
health := cbClient.GetHealth()
if !health.Healthy {
    log.Warn("circuit open", "state", health.State, "failures", health.TotalFailures)
}
```

## Related Standards

- circuit-breaker.md - Generic circuit breaker pattern concepts
- retry-logic.md - Generic retry pattern concepts
- jp-go-errors.md - Error classification for retry detection
- jp-go-config.md - Configuration struct for resilience settings
