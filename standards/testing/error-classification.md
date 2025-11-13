# Error Classification Pattern

Type-safe error checking using `errors.Is` for sentinel errors with consistent logging based on error severity.

## Pattern

```go
import "errors"

if errors.Is(err, context.Canceled) {
    logger.Warn("operation cancelled (expected condition)", "error", err)
    return
}

if errors.Is(err, sql.ErrNoRows) {
    logger.Warn("record not found", "id", recordID)
    return &NotFoundError{Resource: "record", ID: recordID}
}
```

## Why Use This Pattern

- **Type safe**: Works even if error message format changes
- **Idiomatic**: Follows Go 1.13+ best practices
- **Reliable**: Checks actual error types, not string representations
- **Consistent**: Standard approach for expected vs unexpected errors
- **Testable**: Error types can be mocked and verified

## Type-Safe Error Checking

```go
// pipeline/pkg/workers/normaliser/normaliser_worker.go
if errors.Is(err, context.Canceled) {
    w.logger.Warn("context cancelled, stopping worker", "error", err)
    return
}

// ❌ BAD: Brittle string matching
if strings.Contains(err.Error(), "context canceled") {
    w.logger.Warn("context cancelled", "error", err)
    return
}
```

## Error Logging Levels

Use consistent log levels based on error classification:

| Condition | Level | Example |
|-----------|-------|---------|
| Expected/recoverable | WARN | Context cancelled, connection busy, duplicate key |
| Unexpected errors | ERROR | Query failures, marshaling errors, constraint violations |
| Successful operations | DEBUG/INFO | Request completed, batch processed |

```go
// Expected condition: WARN level
if errors.Is(err, context.Canceled) {
    r.logger.Warn("operation cancelled (expected)", "error", err)
    return
}

// Unexpected error: ERROR level
if err != nil {
    r.logger.Error("failed to save record", "error", err, "id", id)
    return fmt.Errorf("failed to save record: %w", err)
}

// Success: DEBUG level
r.logger.Debug("saved record", "id", id, "count", count)
```

## Exception for Third-Party Errors

Use string matching only when third-party libraries don't expose sentinel errors:

```go
// Checking pgx driver errors without exposed sentinels
if strings.Contains(errMsg, "tx is closed") ||
    strings.Contains(errMsg, "conn busy") {
    logger.Warn("expected database condition", "error", errMsg)
    return
}
```

## Guidelines

1. **Use errors.Is first**: Always prefer type-safe error checking
2. **Match log levels**: WARN for expected, ERROR for unexpected
3. **Add context**: Include relevant IDs and operation details in logs
4. **Document exceptions**: Comment why string matching is used
5. **Test error paths**: Verify logging and error handling behavior

## Testing Error Classification

```go
var _ = Describe("Error Classification", func() {
    It("should log expected errors at WARN level", func() {
        logBuffer := &bytes.Buffer{}
        logger := slog.New(slog.NewJSONHandler(logBuffer, &slog.HandlerOptions{
            Level: slog.LevelDebug,
        }))

        // Trigger cancellation
        ctx, cancel := context.WithCancel(context.Background())
        cancel()

        err := worker.Process(ctx)

        logOutput := logBuffer.String()
        Expect(logOutput).To(ContainSubstring(`"level":"WARN"`))
        Expect(logOutput).To(ContainSubstring("cancelled"))
    })
})
```

## Related Patterns

- Error Wrapping: See `.ai/common-standards/error-wrapping.md`
- Transaction Rollback: See `.ai/project-standards/database-transactions.md`

## References

- Go 1.13 Errors: <https://go.dev/blog/go1.13-errors>
- Structured Logging: <https://pkg.go.dev/log/slog>
