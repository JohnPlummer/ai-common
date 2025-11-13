# Test Timeout Constants

Standardized timeout constants for test operations across all test suites.

## Pattern

```go
var _ = Describe("Integration Test", func() {
    BeforeEach(func() {
        SetDefaultEventuallyTimeout(testutil.IntegrationTestTimeout)
    })

    It("should complete within timeout", func() {
        Eventually(func() bool {
            return checkCondition()
        }).Should(BeTrue())
    })
})
```

## Why Use This Pattern

- **Consistency**: Same timeout values across test suites
- **Clarity**: Named constants describe intent
- **Maintainability**: Single source of truth for timeout adjustments
- **Test categorization**: Timeout value indicates test type (unit vs integration vs E2E)

## Available Constants

### Suite-level timeouts

```go
testutil.SuiteSetupTimeout    = 2 * time.Minute  // Container startup, migrations
testutil.SuiteTeardownTimeout = 1 * time.Minute  // Cleanup operations
```

### Test-level timeouts

```go
testutil.UnitTestTimeout        = 10 * time.Second  // Fast, isolated tests
testutil.IntegrationTestTimeout = 30 * time.Second  // Database/external deps
testutil.ComplexTestTimeout     = 1 * time.Minute   // Multi-step operations
testutil.E2ETestTimeout         = 2 * time.Minute   // End-to-end workflows
```

### Operation-level timeouts

```go
testutil.DatabaseOperationTimeout = 15 * time.Second
testutil.APICallTimeout           = 10 * time.Second
testutil.ContainerStartupTimeout  = 45 * time.Second
testutil.RetryOperationTimeout    = 20 * time.Second
```

### Async operation timeouts

```go
testutil.EventuallyTimeout    = 30 * time.Second        // Eventually default
testutil.EventuallyPolling    = 500 * time.Millisecond  // Poll interval
testutil.ConsistentlyDuration = 5 * time.Second         // Consistently default
testutil.ConsistentlyPolling  = 100 * time.Millisecond
```

### Cleanup timeouts

```go
testutil.CleanupTimeout     = 10 * time.Second
testutil.GracePeriodTimeout = 5 * time.Second
```

## Usage Patterns

### Ginkgo default timeouts

```go
BeforeEach(func() {
    SetDefaultEventuallyTimeout(testutil.IntegrationTestTimeout)
    SetDefaultEventuallyPollingInterval(testutil.EventuallyPolling)
})
```

### Context timeouts

```go
ctx, cancel := context.WithTimeout(context.Background(), testutil.APICallTimeout)
defer cancel()
```

### Inline Eventually

```go
Eventually(func() bool {
    return condition()
}, testutil.IntegrationTestTimeout, testutil.EventuallyPolling).Should(BeTrue())
```

## Guidelines

1. **Use appropriate constant**: Match timeout to operation type
2. **Don't override without reason**: Constants provide consistency
3. **Extend for slow operations**: Some operations may need longer
4. **Document exceptions**: Comment why custom timeout needed

## Related Patterns

- `.ai/project-standards/test-suite-logging.md` - Test suite configuration
