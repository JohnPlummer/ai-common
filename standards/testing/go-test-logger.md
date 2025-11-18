# go-test-logger Package Usage

*Category: testing*

Test logging utilities for Ginkgo/Gomega that hide expected error logs while validating patterns.

## When to Use

Use go-test-logger for clean Ginkgo/Gomega tests:

- Hiding expected error logs from test output (ExpectErrorLog pattern)
- Suite-level logging configuration with LOG_LEVEL environment variable
- Validating structured log output (JSON and text formats)
- Negative assertions (ensuring no errors occurred)

## Package Overview

go-test-logger provides:

- ExpectErrorLog() - Captures and validates expected errors without polluting output
- ConfigureTestLogging() - Suite-level slog configuration
- WithCapturedLogger() - Manual log capture with Gomega buffers
- AssertNoErrorLogs() - Validates successful operations

Integrates with log/slog and Gomega's gbytes for pattern matching.

## Import Convention

```go
import testlogger "github.com/JohnPlummer/go-test-logger"
```

## ExpectErrorLog Pattern

Hides expected errors from test output while validating they occurred:

```go
It("should handle API rate limits", func() {
    testlogger.ExpectErrorLog(func(logger *slog.Logger) {
        client := NewClient(logger)
        err := client.CallAPI() // Logs "rate limit exceeded"
        Expect(err).To(HaveOccurred())
    }, "rate limit exceeded", "status=429")
})
```

**How it works:**

- Captures logs to in-memory buffer
- Validates expected patterns appear
- Hides matching logs from output
- Displays unexpected logs to stderr

**JSON format variant:**

```go
testlogger.ExpectErrorLogJSON(func(logger *slog.Logger) {
    service.ProcessInvalidData()
}, `"level":"ERROR"`, `"msg":"validation failed"`, `"field":"email"`)
```

## Suite-Level Logging Configuration

Configure slog defaults in BeforeSuite:

```go
var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()
    // Suite setup continues...
})
```

**Default behavior:**

- Suppresses INFO and WARN (reduces noise)
- Shows ERROR logs to stderr (for debugging)
- Writes to stderr (not stdout)

**LOG_LEVEL environment variable:**

```bash
LOG_LEVEL=DEBUG ginkgo run ./...  # See all logs
LOG_LEVEL=ERROR ginkgo run ./...  # Errors only (default)
```

Values: `DEBUG`, `INFO`, `WARN`, `ERROR`

## Manual Log Capture

For custom validation patterns:

```go
It("should log processing steps", func() {
    logger, buffer := testlogger.WithCapturedLogger(slog.LevelInfo)
    service := NewService(logger)
    service.ProcessData()

    Expect(buffer).To(gbytes.Say("processing started"))
    Expect(buffer).To(gbytes.Say("processing completed"))
})
```

**JSON variant:**

```go
logger, buffer := testlogger.WithCapturedJSONLogger(slog.LevelInfo)
handler.HandleRequest(req)
Expect(buffer).To(gbytes.Say(`"request_id":"123"`))
```

## Negative Assertions

Validate no errors occurred:

```go
It("should process valid data without errors", func() {
    logger, buffer := testlogger.WithCapturedLogger(slog.LevelDebug)
    service := NewService(logger)

    service.ProcessValidData()

    testlogger.AssertNoErrorLogs(buffer)
})
```

Checks for both text and JSON error formats.

## Integration with Ginkgo

Complete test suite example:

```go
package mypackage_test

import (
    "testing"
    testlogger "github.com/JohnPlummer/go-test-logger"
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
)

func TestMyPackage(t *testing.T) {
    RegisterFailHandler(Fail)
    RunSpecs(t, "MyPackage Suite")
}

var _ = BeforeSuite(func() {
    testlogger.ConfigureTestLogging()
})

var _ = Describe("API Client", func() {
    It("handles expected errors", func() {
        testlogger.ExpectErrorLog(func(logger *slog.Logger) {
            // Test code using logger
        }, "expected pattern")
    })
})
```

## Related Standards

- expected-error-logs.md - Pattern explanation and rationale
- test-suite-logging.md - Suite configuration patterns
- bdd-testing.md - Ginkgo/Gomega best practices
