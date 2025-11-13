# Test Suite Logging Configuration

Configure test logging to suppress noise while preserving error visibility.

## Pattern

```go
var _ = BeforeSuite(func() {
    testutil.ConfigureTestLogging()

    // Suite setup continues...
})
```

## Why Use This Pattern

- **Clean output**: Suppresses INFO and WARN logs during test runs
- **Error visibility**: ERROR logs still appear for debugging
- **Environment control**: Respects LOG_LEVEL env var for debugging specific suites
- **Consistent behavior**: Same logging config across all test suites

## Examples from Codebase

### Integration test suite setup

```go
// pipeline/tests/integration/repository/repository_suite_test.go:35
var _ = BeforeSuite(func() {
    testutil.ConfigureTestLogging()

    ctx = context.Background()
    pgTestContainer, err = containers.StartSimplePostgreSQLContainer(ctx)
    Expect(err).NotTo(HaveOccurred())
})
```

### Unit test suite setup

```go
// pipeline/pkg/llm/llm_suite_test.go:19
var _ = BeforeSuite(func() {
    testutil.ConfigureTestLogging()
})
```

### Backend integration test

```go
// backend/pkg/api/repositories/repository_integration_test.go:29
var _ = BeforeSuite(func() {
    testutil.ConfigureTestLogging()

    if shouldSkip, skipMessage := containers.SkipIfDockerUnavailable(); shouldSkip {
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

## Implementation

From `shared/testutil/logging.go:16`:

- Default: `slog.Level(7)` (just below ERROR)
- Outputs to stderr for visibility
- Sets as default logger for entire suite

## Related Patterns

- `.ai/project-standards/expected-error-logs.md` - Validating expected errors
