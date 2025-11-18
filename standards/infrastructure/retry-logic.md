# Retry Logic Pattern

*Category: infrastructure*

Handles transient failures with exponential backoff and jitter to prevent overwhelming failing services.

## Pattern Overview

Retry logic automatically retries failed operations that might succeed on subsequent attempts. Uses exponential backoff (doubling delays) with jitter (random variance) to prevent thundering herd problems.

**For implementation**, use jp-go-resilience package (see `go/jp-go-resilience.md`).

## Why Use This Pattern

- **Resilience**: Handles transient failures (network blips, rate limits, temporary outages)
- **Exponential backoff**: Increasingly longer delays prevent overwhelming failing services
- **Jitter**: Random variance prevents thundering herd when multiple clients retry
- **Smart failure detection**: Only retries errors marked as retryable
- **Context aware**: Respects cancellation and timeouts

## When to Retry

Retry is appropriate for transient (temporary) failures:

**Retry these errors:**

- Network timeouts and temporary unavailability
- Rate limiting (HTTP 429)
- Service temporarily unavailable (HTTP 503)
- Database deadlocks and connection timeouts

**Never retry these errors:**

- Validation errors (HTTP 400)
- Authentication/authorization errors (HTTP 401, 403)
- Not found errors (HTTP 404)
- Internal logic errors (programming bugs)

The jp-go-resilience package integrates with jp-go-errors.IsRetryable() to automatically classify errors.

## Backoff Strategies

**Exponential** (recommended): Delays double each retry (1s → 2s → 4s)

- Best for most API calls
- Quickly backs off to avoid overwhelming services

**Constant**: Fixed delay between retries (2s → 2s → 2s)

- Use for predictable retry windows
- Good for operations with known recovery times

**Fibonacci**: Delays follow Fibonacci sequence (1s → 2s → 3s → 5s)

- Moderate growth between exponential and constant

## Combining with Circuit Breakers

Layering retry inside circuit breaker provides optimal resilience:

- Circuit breaker fails fast when service is known to be down
- Retry handles occasional transient failures
- Prevents wasted retries to dead services

See jp-go-resilience.md for CombineRetryAndCircuitBreaker() implementation.

## Related Standards

- `go/jp-go-resilience.md` - Implementation with retry wrapper
- `infrastructure/circuit-breaker.md` - Combining with circuit breakers
- `go/jp-go-errors.md` - Error classification for retry detection
