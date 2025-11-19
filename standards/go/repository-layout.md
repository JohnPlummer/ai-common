# Go Repository Layout

Standard directory structure for Go projects.

## Core Structure

```
project/
├── cmd/              # Executables (one dir per binary)
│   └── myapp/
│       └── main.go   # Minimal main, imports from internal/
├── internal/         # Private application code (default)
├── bin/             # Compiled binaries (gitignored)
├── pkg/             # Public libraries (rarely needed)
├── tests/           # Integration/e2e tests
├── migrations/      # Database migrations
└── tmp/            # Temporary test files (gitignored)
```

## Decision Tree

**Where does my code go?**

```
Is it an executable?
├─ Yes → cmd/appname/main.go
└─ No → Will other projects import it?
    ├─ Maybe later → internal/
    ├─ Yes, stable API → pkg/ (or separate module)
    └─ No → internal/
```

**Do I need pkg/?** Usually NO.

Use `pkg/` only if:

- Creating library for external use AND
- API is stable and documented AND
- Committed to maintaining compatibility

Otherwise: Start with `internal/`, extract later if needed.

## Rules

### cmd/

- Each subdirectory produces one binary
- Keep main.go minimal (wiring only)
- Business logic lives in `internal/`

```go
// cmd/worker/main.go
func main() {
    cfg := config.LoadFromEnv()
    worker := worker.New(cfg)
    worker.Run()
}
```

### internal/

- All application-specific code (default choice)
- Compiler enforces import restrictions
- Cannot be imported by other projects

### bin/

- All compiled executables
- Add to `.gitignore`
- Makefile: `go build -o bin/myapp ./cmd/myapp`

### pkg/

- **USE SPARINGLY**
- Only for stable, public libraries
- Better: Extract to separate module

## Anti-patterns

❌ Business logic in cmd/

```go
func main() {
    // 500 lines here... NO
}
```

❌ Binaries in source dirs

```
internal/worker/worker  # Binary mixed with source
```

❌ Empty pkg/ directory

```
pkg/myapp/stuff.go  # Don't nest under project name
```

❌ Deep nesting

```
internal/app/domain/services/worker/impl/  # Too deep
```

## Pipeline Example

```
pipeline/
├── cmd/
│   ├── ingestor/main.go
│   ├── scorer/main.go
│   └── worker/main.go
├── internal/          # All implementation
│   ├── cleanup/
│   ├── orchestrator/
│   └── repository/
├── bin/              # Build output
├── tests/
│   ├── integration/
│   └── e2e/
└── tmp/
```

**Why no pkg/?**

- Pipeline-specific code
- Not for external projects
- Can promote later if needed

## References

- [Go Project Layout](https://github.com/golang-standards/project-layout)
