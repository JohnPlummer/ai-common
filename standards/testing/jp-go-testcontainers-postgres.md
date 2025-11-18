# jp-go-testcontainers-postgres Package Usage

*Category: testing*

PostgreSQL testcontainer utilities for Ginkgo/Gomega integration tests with automatic migration support and cleanup patterns.

## When to Use

Use jp-go-testcontainers-postgres for integration tests:

- Repository layer testing with real PostgreSQL
- Testing migrations and schema changes
- Multi-test isolation with table cleanup
- PostGIS spatial query testing
- Services requiring database interactions

## Package Overview

jp-go-testcontainers-postgres wraps testcontainers-go/modules/postgres with:

- One-line container setup with sensible defaults
- Automatic migration detection and running via golang-migrate/migrate
- PostGIS 3.4 support for spatial queries (ST_DWithin, ST_MakePoint)
- Table cleanup patterns for test isolation
- Docker availability checks with graceful skipping
- Connection pool management (pgxpool)

## Import Convention

```go
import testpg "github.com/JohnPlummer/jp-go-testcontainers-postgres"
```

## Basic Container Usage

Simple setup for most tests:

```go
var _ = BeforeSuite(func() {
    ctx := context.Background()
    container, err := testpg.StartSimplePostgreSQLContainer(ctx)
    Expect(err).NotTo(HaveOccurred())
    DeferCleanup(container.WithCleanup())

    // Use container.Pool for database operations
    repo := NewUserRepository(container.Pool)
})
```

**With Docker availability check:**

```go
var _ = BeforeSuite(func() {
    if shouldSkip, msg := testpg.SkipIfDockerUnavailable(); shouldSkip {
        Skip(msg)
    }
    // Continue with container setup
})
```

**Custom configuration:**

```go
config := &testpg.PostgreSQLConfig{
    DatabaseName:      "integration_test",
    Username:          "testuser",
    Password:          "testpass",
    PostgreSQLVersion: "16-3.4", // PostGIS 3.4 + PostgreSQL 16
    MaxConns:          20,
    StartupTimeout:    60 * time.Second,
}
container, err := testpg.StartPostgreSQLContainer(ctx, config)
```

## Table Cleanup Patterns

Isolate tests by cleaning tables between runs:

**Clean specific tables:**

```go
var _ = Describe("User Repository", func() {
    BeforeEach(func() {
        defer container.WithTableCleanup("users", "sessions")()
    })

    It("creates user", func() {
        user := &User{Email: "test@example.com"}
        err := repo.Create(ctx, user)
        Expect(err).NotTo(HaveOccurred())
    })
})
```

**Clean all tables:**

```go
AfterEach(func() {
    err := container.CleanAllTables(ctx)
    Expect(err).NotTo(HaveOccurred())
})
```

**Pattern choice:**

- CleanSpecificTables: Fast, explicit, recommended for most tests
- CleanAllTables: Comprehensive but slower, excludes schema_migrations and PostGIS system tables

## Migration Support

Automatic migration detection and running:

```go
container, err := testpg.StartPostgreSQLContainerWithMigrations(
    ctx,
    "database/migrations", // Relative or absolute path
)
```

**Auto-detection with MIGRATIONS_PATH:**

```bash
MIGRATIONS_PATH=database/migrations ginkgo run ./...
```

**Auto-detection fallback:**

Package searches for migrations from test file location:

- `database/migrations`
- `migrations`
- `sql/migrations`
- `db/migrations`

Uses `.git` directory to find project root in monorepos.

## Error Handling

Specific error types for test debugging:

```go
container, err := testpg.StartPostgreSQLContainerWithCheck(ctx, config)
if errors.Is(err, testpg.ErrDockerNotAvailable) {
    Skip("Docker not running")
}
if errors.Is(err, testpg.ErrMigrationsFailed) {
    Fail("Check migration files")
}
```

Error types:

- ErrDockerNotAvailable
- ErrContainerStartTimeout
- ErrContainerPortConflict
- ErrDatabaseConnFailed
- ErrMigrationsFailed

## Integration with Ginkgo

Complete suite example:

```go
package repository_test

import (
    "context"
    "testing"
    testpg "github.com/JohnPlummer/jp-go-testcontainers-postgres"
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
)

var (
    container *testpg.PostgreSQLTestContainer
    ctx       context.Context
)

func TestRepository(t *testing.T) {
    RegisterFailHandler(Fail)
    RunSpecs(t, "Repository Suite")
}

var _ = BeforeSuite(func() {
    if shouldSkip, msg := testpg.SkipIfDockerUnavailable(); shouldSkip {
        Skip(msg)
    }

    ctx = context.Background()
    var err error
    container, err = testpg.StartPostgreSQLContainerWithMigrations(ctx, "database/migrations")
    Expect(err).NotTo(HaveOccurred())
    DeferCleanup(container.WithCleanup())
})

var _ = Describe("User Repository", func() {
    var repo *UserRepository

    BeforeEach(func() {
        repo = NewUserRepository(container.Pool)
        DeferCleanup(container.WithTableCleanup("users"))
    })

    It("creates and retrieves user", func() {
        // Test implementation
    })
})
```

## Related Standards

- bdd-testing.md - Ginkgo/Gomega test organization
- test-suite-logging.md - Logging during integration tests
- repository-pattern.md - Database layer testing patterns
