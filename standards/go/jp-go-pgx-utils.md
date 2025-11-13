# jp-go-pgx-utils Package Usage

*Category: go*

## When to Use

Use jp-go-pgx-utils when you need PostgreSQL database access with:

- Connection pooling and lifecycle management
- Health checks and monitoring
- Transaction helpers
- Database migrations
- Testcontainer integration
- Consistent error handling

## Package Overview

jp-go-pgx-utils wraps pgx/v5 with common patterns for connection pooling, health checks, and lifecycle management.

Repository: `github.com/JohnPlummer/jp-go-pgx-utils`

## Import Convention

```go
import (
    pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"
    goconfig "github.com/JohnPlummer/jp-go-config"
)
```

Use the `pgxutils` alias for clarity and consistency across services.

## Creating a Connection

### Basic Usage

```go
import (
    pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"
    goconfig "github.com/JohnPlummer/jp-go-config"
)

func main() {
    cfg := &goconfig.DatabaseConfig{
        Host:     "localhost",
        Port:     5432,
        User:     "appuser",
        Password: "apppass",
        Database: "production_db",
        SSLMode:  "require",
        MaxConns: 25,
    }

    db, err := pgxutils.NewConnection(cfg)
    if err != nil {
        log.Fatalf("failed to create database connection: %v", err)
    }
    defer db.Close()

    // Use db for queries
}
```

### With Configuration Validation

```go
func setupDatabase() (*pgxutils.Connection, error) {
    cfg := &goconfig.DatabaseConfig{
        Host:     viper.GetString("database.host"),
        Port:     viper.GetInt("database.port"),
        User:     viper.GetString("database.user"),
        Password: viper.GetString("database.password"),
        Database: viper.GetString("database.name"),
        SSLMode:  "require",
        MaxConns: 25,
    }

    if err := cfg.Validate(); err != nil {
        return nil, fmt.Errorf("invalid database config: %w", err)
    }

    db, err := pgxutils.NewConnection(cfg)
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }

    return db, nil
}
```

## Connection Type

The Connection type provides the core database interface:

```go
type Connection interface {
    // Standard query methods
    Query(ctx context.Context, sql string, args ...interface{}) (pgx.Rows, error)
    QueryRow(ctx context.Context, sql string, args ...interface{}) pgx.Row
    Exec(ctx context.Context, sql string, args ...interface{}) (pgconn.CommandTag, error)

    // Transaction support
    Begin(ctx context.Context) (pgx.Tx, error)
    BeginTx(ctx context.Context, txOptions pgx.TxOptions) (pgx.Tx, error)

    // Connection pooling
    Acquire(ctx context.Context) (*pgxpool.Conn, error)
    WithConnection(ctx context.Context, fn func(*pgxpool.Conn) error) error
    WithTransaction(ctx context.Context, fn func(pgx.Tx) error) error

    // Health and metrics
    IsHealthy(ctx context.Context) bool
    DetailedHealth(ctx context.Context) HealthStatus
    GetMetrics() PoolMetrics

    // Lifecycle
    Close()
}
```

## Backward Compatibility

For code migrating from database.DB:

```go
import (
    database "github.com/JohnPlummer/jp-go-pgx-utils"
)

// Use type alias
type DB = database.Connection

// Use wrapper function
db, err := database.NewDB(cfg)
```

This maintains existing code patterns during migration.

## Query Patterns

### Simple Query

```go
func GetUser(ctx context.Context, db pgxutils.Connection, id string) (*User, error) {
    var user User

    sql := "SELECT id, name, email FROM users WHERE id = $1"
    err := db.QueryRow(ctx, sql, id).Scan(&user.ID, &user.Name, &user.Email)

    if errors.Is(err, pgx.ErrNoRows) {
        return nil, pkgerrors.NewNotFoundError("user", "id", id)
    }

    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }

    return &user, nil
}
```

### Multiple Rows

```go
func ListUsers(ctx context.Context, db pgxutils.Connection, limit int) ([]*User, error) {
    sql := "SELECT id, name, email FROM users LIMIT $1"
    rows, err := db.Query(ctx, sql, limit)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    defer rows.Close()

    var users []*User
    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Name, &user.Email); err != nil {
            return nil, fmt.Errorf("scan failed: %w", err)
        }
        users = append(users, &user)
    }

    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("rows error: %w", err)
    }

    return users, nil
}
```

### Execute Statement

```go
func UpdateUser(ctx context.Context, db pgxutils.Connection, user *User) error {
    sql := "UPDATE users SET name = $1, email = $2 WHERE id = $3"
    _, err := db.Exec(ctx, sql, user.Name, user.Email, user.ID)

    if err != nil {
        return fmt.Errorf("update failed: %w", err)
    }

    return nil
}
```

## Transaction Patterns

### Using WithTransaction Helper

```go
func TransferFunds(ctx context.Context, db pgxutils.Connection, from, to string, amount decimal.Decimal) error {
    return db.WithTransaction(ctx, func(tx pgx.Tx) error {
        // Debit source account
        if _, err := tx.Exec(ctx, "UPDATE accounts SET balance = balance - $1 WHERE id = $2", amount, from); err != nil {
            return fmt.Errorf("debit failed: %w", err)
        }

        // Credit destination account
        if _, err := tx.Exec(ctx, "UPDATE accounts SET balance = balance + $1 WHERE id = $2", amount, to); err != nil {
            return fmt.Errorf("credit failed: %w", err)
        }

        return nil
    })
}
```

The transaction automatically rolls back on error and commits on success.

### Manual Transaction Control

```go
func ComplexOperation(ctx context.Context, db pgxutils.Connection) error {
    tx, err := db.Begin(ctx)
    if err != nil {
        return fmt.Errorf("begin transaction failed: %w", err)
    }
    defer tx.Rollback(ctx)  // Rollback if not committed

    // Perform operations
    if _, err := tx.Exec(ctx, "INSERT INTO logs ..."); err != nil {
        return err
    }

    if _, err := tx.Exec(ctx, "UPDATE stats ..."); err != nil {
        return err
    }

    // Commit transaction
    if err := tx.Commit(ctx); err != nil {
        return fmt.Errorf("commit failed: %w", err)
    }

    return nil
}
```

## Health Checks

### Simple Health Check

```go
func CheckDatabaseHealth(ctx context.Context, db pgxutils.Connection) error {
    if !db.IsHealthy(ctx) {
        return fmt.Errorf("database is unhealthy")
    }
    return nil
}
```

### Detailed Health Status

```go
func GetDatabaseStatus(ctx context.Context, db pgxutils.Connection) {
    health := db.DetailedHealth(ctx)

    log.Printf("Database Health:")
    log.Printf("  Healthy: %v", health.Healthy)
    log.Printf("  Available Connections: %d", health.AvailableConnections)
    log.Printf("  Idle Connections: %d", health.IdleConnections)
    log.Printf("  Total Connections: %d", health.TotalConnections)
    log.Printf("  Response Time: %v", health.ResponseTime)
}
```

## Connection Pool Metrics

```go
func MonitorConnectionPool(db pgxutils.Connection) {
    metrics := db.GetMetrics()

    log.Printf("Pool Metrics:")
    log.Printf("  Acquire Count: %d", metrics.AcquireCount)
    log.Printf("  Acquire Duration: %v", metrics.AcquireDuration)
    log.Printf("  Acquired Connections: %d", metrics.AcquiredConns)
    log.Printf("  Idle Connections: %d", metrics.IdleConns)
    log.Printf("  Max Connections: %d", metrics.MaxConns)
}
```

## Testing with Testcontainers

```go
import (
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"
    goconfig "github.com/JohnPlummer/jp-go-config"
)

var (
    testDB        pgxutils.Connection
    testContainer *postgres.PostgresContainer
)

var _ = BeforeSuite(func() {
    ctx := context.Background()

    // Start PostgreSQL container
    container, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:16-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("testuser"),
        postgres.WithPassword("testpass"),
    )
    Expect(err).NotTo(HaveOccurred())
    testContainer = container

    // Get connection details
    host, _ := container.Host(ctx)
    port, _ := container.MappedPort(ctx, "5432")

    // Create test database config
    cfg := &goconfig.DatabaseConfig{
        Host:     host,
        Port:     port.Int(),
        User:     "testuser",
        Password: "testpass",
        Database: "testdb",
        SSLMode:  "disable",
        MaxConns: 10,
    }

    // Connect to test database
    testDB, err = pgxutils.NewConnection(cfg)
    Expect(err).NotTo(HaveOccurred())
})

var _ = AfterSuite(func() {
    if testDB != nil {
        testDB.Close()
    }
    if testContainer != nil {
        testContainer.Terminate(context.Background())
    }
})
```

## Database Migrations

```go
import "github.com/golang-migrate/migrate/v4"

func RunMigrations(db pgxutils.Connection, migrationsPath string) error {
    // jp-go-pgx-utils provides migration helpers
    if err := db.RunMigrations(migrationsPath); err != nil {
        return fmt.Errorf("migrations failed: %w", err)
    }
    return nil
}
```

## Error Handling

jp-go-pgx-utils preserves underlying pgx errors for type assertions:

```go
import "github.com/jackc/pgx/v5/pgconn"

_, err := db.Exec(ctx, "INSERT INTO users (id, email) VALUES ($1, $2)", id, email)
if err != nil {
    // Check for specific PostgreSQL errors
    var pgErr *pgconn.PgError
    if errors.As(err, &pgErr) {
        if pgErr.Code == "23505" {  // unique_violation
            return pkgerrors.NewConflictError("user with email already exists")
        }
    }
    return fmt.Errorf("insert failed: %w", err)
}
```

## Related Standards

- jp-go-config.md - DatabaseConfig for connection configuration
- jp-go-errors.md - Error handling patterns
- database-transactions.md - Transaction best practices
- testcontainers.md - Testing with testcontainers

## Common Pitfalls

1. **Always use context** - Pass context to all database operations:

   ```go
   // Wrong
   db.Query("SELECT ...")

   // Right
   db.Query(ctx, "SELECT ...")
   ```

2. **Always close rows** - Defer rows.Close() immediately:

   ```go
   rows, err := db.Query(ctx, sql)
   if err != nil {
       return err
   }
   defer rows.Close()  // Must close
   ```

3. **Check rows.Err()** - After iterating rows:

   ```go
   for rows.Next() {
       // scan
   }
   if err := rows.Err(); err != nil {
       return err
   }
   ```

4. **Use prepared statements for repeated queries** - Cache when possible
5. **Don't ignore context cancellation** - Respect ctx.Done()
6. **Always defer tx.Rollback()** - Even when committing manually
