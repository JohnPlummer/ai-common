# Database Transaction Patterns

Standard patterns for transaction management including shared rollback utilities, configurable timeouts, and batch size validation.

## Pattern

### Using WithTransaction Helper (Recommended)

```go
import pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"

func (r *repository) ProcessWithTransaction(ctx context.Context, items []Item) error {
    return r.db.WithTransaction(ctx, func(tx pgx.Tx) error {
        // Perform operations...
        // Automatic rollback on error, commit on success
        for _, item := range items {
            if err := r.processItem(ctx, tx, item); err != nil {
                return fmt.Errorf("failed to process item: %w", err)
            }
        }
        return nil
    })
}
```

### Manual Transaction Management

```go
func (r *repository) ProcessWithTransaction(ctx context.Context) error {
    txCtx, cancel := context.WithTimeout(ctx, r.transactionTimeout)
    defer cancel()

    tx, err := r.db.Begin(txCtx)
    if err != nil {
        return fmt.Errorf("failed to begin transaction: %w", err)
    }
    defer tx.Rollback(txCtx)  // Rollback if not committed

    // Perform operations...

    if err = tx.Commit(txCtx); err != nil {
        return fmt.Errorf("failed to commit: %w", err)
    }
    return nil
}
```

## Why Use This Pattern

- **DRY**: Eliminates duplicate rollback logic across repositories
- **Consistent**: All components use same error classification
- **Safe**: Automatic rollback on errors with timeout protection
- **Testable**: Single place to test rollback behavior
- **Maintainable**: Changes to rollback logic happen once

## Transaction Helper Options

### Option 1: Use WithTransaction Helper (Recommended)

jp-go-pgx-utils provides a `WithTransaction` helper that handles rollback automatically:

```go
import pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"

func (r *repository) ProcessItems(ctx context.Context, items []Item) error {
    return r.db.WithTransaction(ctx, func(tx pgx.Tx) error {
        // Operations here
        // Automatic rollback on error, commit on success
        for _, item := range items {
            if _, err := tx.Exec(ctx, "INSERT INTO items ...", item); err != nil {
                return err
            }
        }
        return nil
    })
}
```

### Option 2: Shared Rollback Utility (Custom)

Create a shared rollback utility in your project for consistent error handling:

```go
// pkg/database/rollback.go
func HandleTransactionRollback(ctx context.Context, tx pgx.Tx, logger *slog.Logger) {
    if err := tx.Rollback(ctx); err != nil {
        if errors.Is(err, pgx.ErrTxClosed) {
            // Expected - transaction already closed
            return
        }
        logger.Error("failed to rollback transaction", "error", err)
    }
}

// Usage
defer func() {
    if err != nil {
        HandleTransactionRollback(ctx, tx, r.logger)
    }
}()
```

**Benefits:**

- Consistent error classification for expected vs unexpected errors
- Proper logging at appropriate levels
- Handles context cancellation gracefully

## Transaction Timeouts

Always use configurable timeouts to prevent indefinite hangs:

```go
import (
    pgxutils "github.com/JohnPlummer/jp-go-pgx-utils"
)

type repository struct {
    db                 pgxutils.Connection
    logger             *slog.Logger
    transactionTimeout time.Duration
}

func WithTransactionTimeout(timeout time.Duration) RepositoryOption {
    return func(r *repository) {
        r.transactionTimeout = timeout
    }
}

func NewRepository(db pgxutils.Connection, opts ...RepositoryOption) *repository {
    r := &repository{
        db:                 db,
        logger:             slog.Default(),
        transactionTimeout: 30 * time.Second, // Default
    }
    for _, opt := range opts {
        opt(r)
    }
    return r
}
```

**Usage:**

```go
// Create timeout context before Begin()
txCtx, cancel := context.WithTimeout(ctx, r.transactionTimeout)
defer cancel()

tx, err := r.db.Begin(txCtx)
// Use txCtx for all transaction operations
```

## Batch Size Validation

Validate batch sizes to prevent PostgreSQL parameter limit issues:

```go
const maxBatchSize = 1000

func (r *repository) GetBatchForProcessing(ctx context.Context, limit int) ([]*Record, error) {
    if limit > maxBatchSize {
        return nil, fmt.Errorf("batch size %d exceeds maximum %d", limit, maxBatchSize)
    }
    if limit <= 0 {
        return nil, fmt.Errorf("batch size must be positive, got %d", limit)
    }

    // Proceed with query...
}
```

## Guidelines

1. **Prefer WithTransaction helper**: Use jp-go-pgx-utils' `WithTransaction` for automatic rollback
2. **Create timeout first**: Call `context.WithTimeout` before `Begin()`
3. **Use timeout context**: Pass timeout context to all transaction operations
4. **Validate batch sizes**: Check limits before constructing queries
5. **Defer cancellation**: Always defer `cancel()` after creating timeout context

## Common Mistakes

```go
// ❌ BAD: No timeout protection
tx, err := r.db.Begin(ctx)

// ✅ GOOD: Timeout before Begin()
txCtx, cancel := context.WithTimeout(ctx, r.transactionTimeout)
defer cancel()
tx, err := r.db.Begin(txCtx)

// ❌ BAD: Using original context instead of timeout context
result, err := tx.Exec(ctx, query)

// ✅ GOOD: Using timeout context
result, err := tx.Exec(txCtx, query)
```

## Related Patterns

- jp-go-pgx-utils.md: Database connection and WithTransaction helper
- repository-pattern.md: Repository pattern implementation
- error-classification.md: Classifying expected vs unexpected errors

## References

- Go Context: <https://pkg.go.dev/context>
- pgx Transactions: <https://pkg.go.dev/github.com/jackc/pgx/v5>
