# Circuit Breaker Pattern

Circuit breaker pattern using sony/gobreaker for fault tolerance when calling external services.

## Pattern

```go
import "github.com/sony/gobreaker/v2"

// Standard circuit breaker configuration
settings := gobreaker.Settings{
    Name:        "ServiceName",
    MaxRequests: 3,                    // Half-open state requests
    Interval:    10 * time.Second,     // Count window
    Timeout:     30 * time.Second,     // Open to half-open transition
    ReadyToTrip: func(counts gobreaker.Counts) bool {
        failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
        return counts.Requests >= 3 && failureRatio >= 0.6
    },
    OnStateChange: func(name string, from, to gobreaker.State) {
        logger.Info("circuit breaker state changed",
            "name", name,
            "from", from.String(),
            "to", to.String())
    },
}

cb := gobreaker.NewCircuitBreaker[any](settings)

// Execute with circuit breaker
result, err := cb.Execute(func() (any, error) {
    return externalService.Call()
})
```

## Why Use This Pattern

- **Fault tolerance**: Prevents cascading failures when external services fail
- **Fast failure**: Fails fast when service is known to be down
- **Automatic recovery**: Tests service health and automatically recovers
- **Observable**: State changes logged for monitoring
- **Standard library**: Sony gobreaker is industry standard

## Configuration

Standard settings for most use cases:

- **MaxRequests**: 3 (test with 3 requests in half-open state)
- **Interval**: 10s (sliding window for failure counting)
- **Timeout**: 30s (how long to stay open before trying half-open)
- **ReadyToTrip**: 60% failure rate over minimum 3 requests

Adjust based on:

- Service criticality (lower threshold for critical services)
- Expected latency (longer timeout for slow services)
- Traffic volume (higher request minimum for high-traffic)

## Circuit Breaker States

**Closed**: Normal operation, requests pass through

- Tracks failure rate in rolling window
- Trips to Open when threshold exceeded

**Open**: Failing fast, requests rejected immediately

- No requests reach external service
- Transitions to Half-Open after Timeout

**Half-Open**: Testing if service recovered

- Allows MaxRequests through
- Success → Closed, Failure → Open

## Example from Codebase

**Location**: `pipeline/pkg/llm/circuit_breaker.go`

Circuit breaker wrapping OpenAI API calls with logging and metrics.

## Related Patterns

- Combine with retry logic: `.ai/project-standards/retry-logic.md`
- Error classification: `.ai/project-standards/error-classification.md`
