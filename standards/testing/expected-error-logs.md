# Expected Error Log Validation

*Category: testing*

Validate expected error logs appear while suppressing them from test output, catching unexpected errors.

## Import Convention

```go
import testlogger "github.com/JohnPlummer/go-test-logger"
```

## Pattern

```go
It("should handle API errors gracefully", func() {
    testlogger.ExpectErrorLog(func(logger *slog.Logger) {
        client := NewClient(logger)
        err := client.FetchData()

        Expect(err).To(HaveOccurred())
        // Verify error handling behavior

    }, "API request failed", "status=500")
    // ↑ Expected patterns - verified and hidden from output
    // Any OTHER error logs appear in stderr
})
```

## Why Use This Pattern

- **Clean test output**: Expected errors hidden, only unexpected errors shown
- **Safety**: Catches new/unexpected errors immediately
- **Validation**: Asserts expected error logs actually occurred
- **Debugging**: Unexpected errors appear in stderr for investigation

## Examples from Codebase

### Worker error handling

```go
It("logs error and continues processing after batch errors", func() {
    testlogger.ExpectErrorLog(func(logger *slog.Logger) {
        mockProc := &mockBatchProcessor{}
        config := &WorkerConfig{
            PollInterval: 50 * time.Millisecond,
        }
        testWorker, err := NewWorkerWithProcessor(mockProc, config, nil, logger)

        mockProc.processFunc = func(ctx context.Context) (*BatchStatistics, error) {
            return nil, errors.New("batch processing failed")
        }

        // Test continues, verifies worker doesn't crash
    }, "batch processing failed")
})
```

### LLM API error logging

```go
It("logs error details for rate limit errors", func() {
    testlogger.ExpectErrorLog(func(logger *slog.Logger) {
        testExtractor, mockClient := newTestExtractorWithLogger(logger)

        rateLimitErr := &openai.APIError{
            HTTPStatusCode: 429,
            Message:        "You exceeded your current quota",
        }
        mockClient.err = rateLimitErr

        _, err := testExtractor.Extract(ctx, req)
        Expect(err).To(HaveOccurred())
    }, "OpenAI API error details", "Type", "Code")
})
```

### Processor failure handling

```go
It("should mark as failed on generation error", func() {
    testlogger.ExpectErrorLog(func(logger *slog.Logger) {
        client.err = fmt.Errorf("LLM API error")

        processor := NewProcessor(repo, generator, validator, logger)
        err := processor.ProcessActivity(ctx, activity)

        Expect(err).To(HaveOccurred())
        Expect(repo.markFailedCalled).To(BeTrue())
    }, "failed to generate content", "invalid LLM response")
})
```

## Guidelines

1. **Use for intentional error paths**: Tests that trigger expected error logs
2. **Multiple patterns**: Pass all expected error message fragments
3. **Logger injection**: Pass logger to system under test
4. **Verify behavior**: Still assert expected error handling behavior
5. **Unexpected errors visible**: Any other ERROR logs appear in stderr automatically

## Pattern Variations

```go
// Single expected error
testlogger.ExpectErrorLog(func(logger *slog.Logger) {
    // test
}, "error message")

// Multiple validation patterns
testlogger.ExpectErrorLog(func(logger *slog.Logger) {
    // test
}, "pattern1", "pattern2", "pattern3")

// JSON format (for structured logging)
testlogger.ExpectErrorLogJSON(func(logger *slog.Logger) {
    // test
}, `"level":"ERROR"`, `"msg":"failure"`)
```

## How It Works

The go-test-logger package provides this functionality:

1. Captures all logs to in-memory buffer
2. Validates each expected pattern appears (fails test if missing)
3. Displays only log lines NOT matching expected patterns to stderr
4. Result: Expected errors hidden, unexpected errors visible

See `testing/go-test-logger.md` for complete API documentation.

## Related Standards

- `testing/go-test-logger.md` - Package documentation and all available functions
- `testing/test-suite-logging.md` - Suite-level logging configuration
