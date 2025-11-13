# Testcontainers Pattern

Automatic PostgreSQL container provisioning for integration tests. Provides isolated, reproducible database environments.

## Pattern

```go
import "github.com/testcontainers/testcontainers-go/modules/postgres"

var (
    container *postgres.PostgresContainer
    pool      *pgxpool.Pool
)

var _ = BeforeSuite(func() {
    container, err := postgres.Run(ctx,
        "postgres:16-alpine",
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("testuser"),
        postgres.WithPassword("testpass"),
    )
    Expect(err).NotTo(HaveOccurred())

    connStr, _ := container.ConnectionString(ctx)
    pool, err = pgxpool.New(ctx, connStr)
    Expect(err).NotTo(HaveOccurred())

    runMigrations(ctx, connStr)
})

var _ = AfterSuite(func() {
    pool.Close()
    container.Terminate(ctx)
})
```

## Why Use This Pattern

- **Isolation**: Each test suite gets fresh database
- **Consistency**: Identical environment across all machines
- **No external setup**: Docker handles everything
- **Migrations**: Real schema migrations applied
- **CI/CD ready**: Works without external services

## Requirements

- Docker running
- Docker permissions (no sudo on Linux)
- Network access to pull postgres:16-alpine

## Repository Integration Test

```go
// backend/pkg/api/repositories/repository_integration_test.go
var _ = Describe("Repository Integration", func() {
    var (
        pgContainer *containers.PostgreSQLTestContainer
        db          *pgxpool.Pool
        repo        repositories.ActivityRepository
    )

    BeforeEach(func() {
        pgContainer, err = containers.StartPostgreSQLContainerWithMigrations(
            context.Background(),
            "../../database/migrations",
        )
        Expect(err).NotTo(HaveOccurred())

        db = pgContainer.GetPool()
        repo = repositories.NewPostgresActivityRepository(db)
    })

    AfterEach(func() {
        pgContainer.Close()
    })

    It("should insert and retrieve activity", func() {
        activity := &types.Activity{
            ID:    "test-id",
            Title: "Test Activity",
        }

        err := repo.Create(ctx, activity)
        Expect(err).NotTo(HaveOccurred())

        retrieved, err := repo.GetByID(ctx, "test-id")
        Expect(err).NotTo(HaveOccurred())
        Expect(retrieved.Title).To(Equal("Test Activity"))
    })
})
```

## Worker Integration Test

```go
// pipeline/pkg/workers/integration_test.go
var _ = Describe("Worker Integration", func() {
    var (
        container *postgres.PostgresContainer
        pool      *pgxpool.Pool
        worker    *Worker
    )

    BeforeEach(func() {
        container, err := postgres.Run(ctx, "postgres:16-alpine")
        Expect(err).NotTo(HaveOccurred())

        connStr, _ := container.ConnectionString(ctx)
        pool, err = pgxpool.New(ctx, connStr)
        Expect(err).NotTo(HaveOccurred())

        setupSchema(pool)
        repo := NewRepository(pool)
        worker = NewWorker(repo)
    })

    AfterEach(func() {
        pool.Close()
        container.Terminate(ctx)
    })

    It("should process and persist data", func() {
        err := worker.Process(ctx)
        Expect(err).NotTo(HaveOccurred())

        var count int
        err = pool.QueryRow(ctx, "SELECT COUNT(*) FROM items").Scan(&count)
        Expect(err).NotTo(HaveOccurred())
        Expect(count).To(BeNumerically(">", 0))
    })
})
```

## Migration Patterns

```go
var _ = BeforeSuite(func() {
    container, err := postgres.Run(ctx, "postgres:16-alpine")
    Expect(err).NotTo(HaveOccurred())

    connStr, _ := container.ConnectionString(ctx)

    // Run golang-migrate migrations
    m, err := migrate.New(
        "file://../../database/migrations",
        connStr,
    )
    Expect(err).NotTo(HaveOccurred())

    err = m.Up()
    Expect(err).NotTo(HaveOccurred())
})
```

## Database Cleanup

```go
AfterEach(func() {
    err := pgContainer.CleanAllTables(context.Background())
    Expect(err).NotTo(HaveOccurred())
})

// Or clean specific tables
AfterEach(func() {
    err := pgContainer.CleanSpecificTables(
        context.Background(),
        "activities",
        "locations",
    )
    Expect(err).NotTo(HaveOccurred())
})
```

## Guidelines

1. **Docker required**: Tests fail gracefully if Docker unavailable
2. **One container per suite**: Create in BeforeSuite, destroy in AfterSuite
3. **Clean between tests**: Use AfterEach to clean tables
4. **Run migrations**: Apply schema in BeforeSuite
5. **Close resources**: Always close pool and terminate container

## Troubleshooting

**Docker Not Running**

```
Error: Cannot connect to Docker daemon
Solution: Start Docker Desktop or Docker daemon
```

**Timeout During Startup**

```
Error: Container startup timeout
Solution: Increase wait strategy timeout or check Docker resources
```

**Permission Denied (Linux)**

```
Error: Permission denied accessing Docker socket
Solution: Add user to docker group
```

## Related Patterns

- Repository Pattern: See `.ai/project-standards/repository-pattern.md`
- BDD Testing: See `.ai/project-standards/bdd-testing.md`

## References

- Testcontainers Go: <https://golang.testcontainers.org/>
- PostgreSQL module: <https://golang.testcontainers.org/modules/postgres/>
- Project testing guide: `docs/testing-strategy.md`
