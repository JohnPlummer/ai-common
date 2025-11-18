# Circuit Breaker Pattern

*Category: infrastructure*

Prevents cascading failures by failing fast when external services are degraded.

## Pattern Overview

Circuit breakers protect your application from repeatedly calling failing external services. When failure rate exceeds a threshold, the circuit "trips" and fails fast without calling the external service, giving it time to recover.

**For implementation**, use jp-go-resilience package (see `go/jp-go-resilience.md`).

## Why Use This Pattern

- **Fault tolerance**: Prevents cascading failures when external services fail
- **Fast failure**: Rejects requests immediately when service is known to be down
- **Automatic recovery**: Tests service health and automatically recovers when available
- **Observable**: State transitions provide monitoring signals
- **Resource protection**: Prevents wasted resources on failing calls

## Circuit Breaker States

Circuit breakers operate in three states:

**Closed** (Normal Operation)

- Requests pass through to external service
- Tracks failure rate in rolling window
- Trips to Open when threshold exceeded (e.g., 60% failures)

**Open** (Failing Fast)

- Requests rejected immediately without calling external service
- Saves resources and prevents cascading failures
- Transitions to Half-Open after timeout period (e.g., 60 seconds)

**Half-Open** (Testing Recovery)

- Allows limited requests through to test service health
- Success transitions back to Closed
- Failure returns to Open

## When to Use

Circuit breakers are essential for:

- HTTP/gRPC calls to external APIs
- Database connections with potential timeouts
- Third-party service integrations (payment gateways, email services)
- Microservice communication

Skip circuit breakers for:

- Internal function calls without I/O
- Already-reliable services (local file system)
- Operations where you need every attempt (user authentication)

## Related Standards

- `go/jp-go-resilience.md` - Implementation with circuit breaker wrapper
- `infrastructure/retry-logic.md` - Combining retry with circuit breakers
- `go/jp-go-errors.md` - Error classification for failure detection
