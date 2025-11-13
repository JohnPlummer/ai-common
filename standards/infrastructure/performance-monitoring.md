# Performance Monitoring with Prometheus

Track performance in production via runtime metrics, not test assertions with absolute thresholds.

## Pattern

```go
import "github.com/prometheus/client_golang/prometheus"

var operationDuration = prometheus.NewHistogramVec(
    prometheus.HistogramOpts{
        Name:    "operation_duration_seconds",
        Buckets: prometheus.ExponentialBuckets(0.001, 2, 10), // 1ms to 1s
    },
    []string{"operation", "status"},
)

func init() {
    prometheus.MustRegister(operationDuration)
}

func (s *Service) Process(input string) (result *Result, err error) {
    start := time.Now()
    defer func() {
        status := "success"
        if err != nil {
            status = "error"
        }
        operationDuration.WithLabelValues("process", status).Observe(time.Since(start).Seconds())
    }()

    result, err = s.doProcess(input)
    return result, err
}
```

## When to Use

Add metrics to:

- **User-facing operations**: Search, filters, form submissions, page loads
- **External calls**: APIs (Maps, OpenAI), database queries, cache operations
- **Batch operations**: Large datasets, migrations, background jobs
- **Critical paths**: Payments, auth, data consistency

## Why Use This Pattern

- **Production insight**: Track real performance under actual load
- **Alerting**: Automated notifications when performance degrades
- **No CI flakiness**: Removes unreliable timing assertions

## Guidelines

1. **Use histograms for durations**: Track P50/P95/P99 latency
2. **Choose buckets**: Fast (1ms-1s), Medium (100ms-100s), Slow (10ms-40s)
3. **Label by status**: Track success vs error rates
4. **Query P95**: `histogram_quantile(0.95, rate(duration_bucket[5m]))`

## Anti-Pattern: Timing Assertions in Tests

```go
// DON'T: Flaky in CI
start := time.Now()
result := service.Process(data)
Expect(time.Since(start)).To(BeNumerically("<", 5*time.Millisecond))

// DO: Test correctness only
result, err := service.Process(data)
Expect(err).NotTo(HaveOccurred())
```

**Why fails**: Shared CPU, background processes, variable hardware.

**Exception**: Keep minimum timing when testing timeout behavior:

```go
// OK: Verify timeout was respected
Expect(time.Since(start)).To(BeNumerically(">=", 100*time.Millisecond))
```

## Alerting

```yaml
- alert: OperationSlow
  expr: histogram_quantile(0.95, rate(duration_bucket[5m])) > 0.010
  for: 5m
```

## Related

- `.ai/project-standards/test-categories.md#timing-assertions-anti-pattern`
