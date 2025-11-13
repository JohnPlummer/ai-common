# Test Timing Patterns

Patterns for handling test timing: when to use Eventually(), when to avoid sleep entirely, and legitimate sleep use cases.

## Eventually Pattern

```go
// Poll condition until true or timeout (default 1s, 10ms polling)
Eventually(func() bool {
    return condition
}).Should(BeTrue())

// Poll for specific value
Eventually(func() int {
    return worker.GetActiveCount()
}).Should(Equal(3))

// Custom timeout and polling interval
Eventually(func() bool {
    return cache.IsExpired()
}, "2s", "50ms").Should(BeTrue())
```

**Benefits:**

- Adaptive timing - returns immediately when condition met
- Load tolerance - handles slow machines and CI variability
- Better diagnostics - shows expected vs actual values on failure
- Faster execution - typically 10-50ms instead of fixed 100-500ms sleep

## When to Use Eventually vs Sleep

| Use Eventually | Use time.Sleep | Never Use Sleep |
|---------------|----------------|-----------------|
| Worker startup | API rate limit simulation | Mock execution delays |
| State changes | Benchmark timing | Race condition forcing |
| Async operations | Mock server latency | Context timeout "paranoia" |
| Cache expiry | Timeout behavior tests | Before async assertions |
| Channel receives | Time-based calculations | |

**REQUIRED**: Document legitimate sleeps:

```go
// Legitimate sleep: simulating API rate limit
time.Sleep(100 * time.Millisecond)
```

## Guidelines

1. **Timeouts**: Use `testutil.EventuallyTimeout` (30s), `testutil.IntegrationTestTimeout` (60s)
2. **Polling**: Default 10ms works for most cases, 50-100ms for database queries
3. **Return values**: Function passed to Eventually should return the value being tested
4. **No nested assertions**: Don't use Expect/Should inside Eventually function

## Critical Anti-patterns

```go
// BAD: Mock execution delay (mock returns instantly)
worker.EXPECT().Start(mock.Anything).RunAndReturn(func(ctx context.Context) error {
    time.Sleep(10 * time.Millisecond) // Pointless
    return nil
})
// GOOD: No artificial delay
worker.EXPECT().Start(mock.Anything).Return(nil)

// BAD: Race condition timing sleep (unreliable)
for i := 0; i < 50; i++ {
    go func() {
        tracker.IncrementFailure(id)
        time.Sleep(time.Microsecond) // Trying to force contention
    }()
}
// GOOD: Use -race flag
for i := 0; i < 50; i++ {
    go func() {
        tracker.IncrementFailure(id) // Run with: go test -race
    }()
}

// BAD: Fixed delay for async operations
time.Sleep(500 * time.Millisecond)
assertPostStatus(db, "post_id", "scored")
// GOOD: Poll until condition met
Eventually(func() string {
    var status string
    _ = db.QueryRow(ctx, "SELECT status FROM posts WHERE id = $1", "post_id").Scan(&status)
    return status
}, "2s", "50ms").Should(Equal("scored"))

// BAD: Paranoia sleep for context timeout
timeoutCtx, cancel := context.WithTimeout(ctx, 1*time.Nanosecond)
defer cancel()
time.Sleep(1 * time.Millisecond) // Context already expired
result, err := operation(timeoutCtx)
// GOOD: Context timeout fires immediately
timeoutCtx, cancel := context.WithTimeout(ctx, 1*time.Nanosecond)
defer cancel()
result, err := operation(timeoutCtx)
```

## Related Patterns

- Test timeout constants: `.ai/project-standards/test-timeout-constants.md`
- BDD testing: `.ai/project-standards/bdd-testing.md`
- Test categories: `.ai/project-standards/test-categories.md`
