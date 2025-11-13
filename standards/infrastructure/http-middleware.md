# HTTP Middleware Pattern

Standard middleware pattern for wrapping HTTP handlers with cross-cutting concerns like logging, timing, and response capture.

## Pattern

```go
func LoggingMiddleware(logger *slog.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()

            // Wrap ResponseWriter to capture status
            wrapped := &responseWriter{
                ResponseWriter: w,
                statusCode: http.StatusOK,
            }

            // Process request
            next.ServeHTTP(wrapped, r)

            // Log after processing
            logger.Info("http request",
                "method", r.Method,
                "path", r.URL.Path,
                "status", wrapped.statusCode,
                "duration", time.Since(start),
            )
        })
    }
}
```

## Why Use This Pattern

- **Separation of concerns**: Cross-cutting logic isolated from handlers
- **Composable**: Chain multiple middlewares together
- **Reusable**: Same middleware across different routes
- **Testable**: Middleware can be unit tested independently
- **Type safe**: Standard http.Handler interface

## Response Writer Wrapper

```go
type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}

func (rw *responseWriter) Write(b []byte) (int, error) {
    return rw.ResponseWriter.Write(b)
}
```

## Middleware Chain

```go
// backend/cmd/api/main.go
router := chi.NewRouter()

// Apply middlewares in order
router.Use(LoggingMiddleware(logger))
router.Use(RecoveryMiddleware(logger))
router.Use(CORSMiddleware())

// Routes use all middlewares
router.Get("/activities", handleGetActivities)
```

## Recovery Middleware

```go
func RecoveryMiddleware(logger *slog.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            defer func() {
                if err := recover(); err != nil {
                    logger.Error("panic recovered",
                        "error", err,
                        "path", r.URL.Path,
                        "method", r.Method,
                    )

                    w.WriteHeader(http.StatusInternalServerError)
                    json.NewEncoder(w).Encode(map[string]string{
                        "code": "INTERNAL_ERROR",
                        "message": "An internal error occurred",
                    })
                }
            }()

            next.ServeHTTP(w, r)
        })
    }
}
```

## Request ID Middleware

```go
func RequestIDMiddleware() func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            requestID := r.Header.Get("X-Request-ID")
            if requestID == "" {
                requestID = uuid.New().String()
            }

            // Add to context
            ctx := context.WithValue(r.Context(), "request_id", requestID)

            // Add to response headers
            w.Header().Set("X-Request-ID", requestID)

            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}
```

## Guidelines

1. **Return closure**: Middleware returns handler-wrapping function
2. **Call next**: Always invoke `next.ServeHTTP()` unless short-circuiting
3. **Wrap writer**: Use custom ResponseWriter to capture status/response
4. **Log after**: Log request details after processing for accurate timing
5. **Use defer**: Defer cleanup and logging to handle panics

## Testing Middleware

```go
var _ = Describe("LoggingMiddleware", func() {
    It("should log request details", func() {
        logBuffer := &bytes.Buffer{}
        logger := slog.New(slog.NewJSONHandler(logBuffer, nil))

        handler := LoggingMiddleware(logger)(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            w.WriteHeader(http.StatusOK)
        }))

        req := httptest.NewRequest("GET", "/test", nil)
        rec := httptest.NewRecorder()

        handler.ServeHTTP(rec, req)

        logOutput := logBuffer.String()
        Expect(logOutput).To(ContainSubstring("GET"))
        Expect(logOutput).To(ContainSubstring("/test"))
        Expect(logOutput).To(ContainSubstring("200"))
    })
})
```

## Related Patterns

- Error Responses: See `.ai/project-standards/error-responses.md`
- Clean Architecture: See `.ai/project-standards/clean-architecture.md`

## References

- Chi Router: <https://github.com/go-chi/chi>
- Go HTTP Package: <https://pkg.go.dev/net/http>
