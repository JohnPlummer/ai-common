# Test Suite Logging Configuration

*Category: testing*

Configure test logging to suppress noise while preserving error visibility.

## Import Convention

```go
import testlogger "github.com/JohnPlummer/go-test-logger"
```

## Pattern

```go
var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()

    // Suite setup continues...
})
```

## Why Use This Pattern

- **Clean output**: Suppresses INFO and WARN logs during test runs
- **Error visibility**: ERROR logs still appear for debugging
- **Environment control**: Respects LOG_LEVEL env var for debugging specific suites
- **Consistent behavior**: Same logging config across all test suites

## Usage Examples

### Integration test suite setup

```go
var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()

    ctx = context.Background()
    pgTestContainer, err = testpg.StartSimplePostgreSQLContainer(ctx)
    Expect(err).NotTo(HaveOccurred())
})
```

### Unit test suite setup

```go
var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()
})
```

### Integration test with Docker check

```go
var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()

    if shouldSkip, skipMessage := testpg.SkipIfDockerUnavailable(); shouldSkip {
        Skip(skipMessage)
    }
})
```

## Guidelines

1. **Always first in BeforeSuite**: Call before any other suite setup
2. **One call per suite**: Don't call in BeforeEach or individual tests
3. **Default is quiet**: INFO/WARN suppressed, ERROR shown
4. **Debug with LOG_LEVEL**: Set `LOG_LEVEL=DEBUG` to see all logs for specific suite debugging

## Environment Variable

```bash
# Show all logs
LOG_LEVEL=DEBUG ginkgo run ./...

# Show INFO and above
LOG_LEVEL=INFO ginkgo run ./pkg/workers

# Show WARN and above
LOG_LEVEL=WARN ginkgo run ./...

# Show ERROR only (default)
ginkgo run ./...
```

## How It Works

The go-test-logger package provides:

- Default: ERROR level only (suppresses INFO and WARN)
- Outputs to stderr for visibility during test runs
- Respects LOG_LEVEL environment variable for debugging
- Sets slog default logger for entire suite

See `testing/go-test-logger.md` for LOG_LEVEL values and advanced usage.

## Related Standards

- `testing/go-test-logger.md` - Package documentation and all available functions
- `testing/expected-error-logs.md` - Validating expected error patterns
