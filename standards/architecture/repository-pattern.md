# Repository Pattern

Standard data access pattern with public interface and private implementation. All database operations go through repositories.

## Pattern

```go
// Public interface
type Repository interface {
    Get(ctx context.Context, id string) (*Entity, error)
    Save(ctx context.Context, entity *Entity) error
}

// Private implementation
type repository struct {
    pool *pgxpool.Pool
}

// Constructor
func NewRepository(pool *pgxpool.Pool) Repository {
    return &repository{pool: pool}
}
```

## Why Use This Pattern

- **Testable**: Easy to mock for unit tests
- **Abstraction**: Hides database implementation details
- **Centralized**: All data access in one place
- **Transaction support**: Consistent transaction handling
- **Type safe**: Strong typing for database operations

## File Naming

- Interface definition: `repository_interface.go` or in service package
- Implementation: `{entity}_repository.go` (e.g., `activity_repository.go`)
- Integration tests: `{entity}_repository_integration_test.go`

## Basic Repository

```go
// pipeline/pkg/repository/activity_repository.go:19
type ActivityRepository interface {
    GetAll(ctx context.Context) ([]Activity, error)
    Create(ctx context.Context, activity *Activity) error
}

type activityRepository struct {
    pool *pgxpool.Pool
}

func NewActivityRepository(pool *pgxpool.Pool) ActivityRepository {
    return &activityRepository{pool: pool}
}

func (r *activityRepository) GetAll(ctx context.Context) ([]Activity, error) {
    query := `SELECT id, title, description FROM activities ORDER BY created_at DESC`

    rows, err := r.pool.Query(ctx, query)
    if err != nil {
        return nil, fmt.Errorf("failed to query activities: %w", err)
    }
    defer rows.Close()

    var activities []Activity
    for rows.Next() {
        var a Activity
        if err := rows.Scan(&a.ID, &a.Title, &a.Description); err != nil {
            return nil, fmt.Errorf("failed to scan activity: %w", err)
        }
        activities = append(activities, a)
    }

    return activities, nil
}
```

## Transaction Support

```go
type Repository interface {
    Save(ctx context.Context, entity *Entity) error
    SaveWithTx(ctx context.Context, tx pgx.Tx, entity *Entity) error
}

func (r *repository) SaveWithTx(ctx context.Context, tx pgx.Tx, entity *Entity) error {
    if tx == nil {
        return fmt.Errorf("transaction cannot be nil")
    }

    query := `INSERT INTO entities (id, name) VALUES ($1, $2)`
    _, err := tx.Exec(ctx, query, entity.ID, entity.Name)
    return err
}
```

## Error Handling

```go
func (r *repository) GetByID(ctx context.Context, id string) (*Entity, error) {
    if id == "" {
        return nil, fmt.Errorf("id cannot be empty")
    }

    query := `SELECT id, name FROM entities WHERE id = $1`
    var entity Entity

    err := r.pool.QueryRow(ctx, query, id).Scan(&entity.ID, &entity.Name)
    if err != nil {
        if errors.Is(err, pgx.ErrNoRows) {
            return nil, nil  // Not found is not an error
        }
        return nil, fmt.Errorf("failed to get entity %s: %w", id, err)
    }

    return &entity, nil
}
```

## Testing Repositories

```go
var _ = Describe("ActivityRepository", func() {
    var (
        repo      Repository
        container *postgres.PostgresContainer
        pool      *pgxpool.Pool
    )

    BeforeEach(func() {
        container, err := postgres.Run(ctx, "postgres:16-alpine")
        Expect(err).NotTo(HaveOccurred())

        connStr, _ := container.ConnectionString(ctx)
        pool, err = pgxpool.New(ctx, connStr)
        Expect(err).NotTo(HaveOccurred())

        repo = NewRepository(pool)
    })

    AfterEach(func() {
        pool.Close()
        container.Terminate(ctx)
    })

    It("should save and retrieve entity", func() {
        entity := &Entity{ID: "1", Name: "Test"}

        err := repo.Create(ctx, entity)
        Expect(err).NotTo(HaveOccurred())

        retrieved, err := repo.GetByID(ctx, "1")
        Expect(err).NotTo(HaveOccurred())
        Expect(retrieved.Name).To(Equal("Test"))
    })
})
```

## Guidelines

1. **Public interface, private implementation**: Always
2. **Context first parameter**: All methods accept context
3. **Validate inputs**: Check for nil, empty values before queries
4. **Wrap errors**: Add context with fmt.Errorf and %w
5. **Not found ≠ error**: Return (nil, nil) for missing entities
6. **Transaction methods**: Suffix with `WithTx` for transaction variants

## Related Patterns

- Clean Architecture: See `.ai/project-standards/clean-architecture.md`
- Testcontainers: See `.ai/project-standards/testcontainers.md`

## References

- Implementation: `pkg/repository/reddit/post_repository.go`
- Testing guide: `docs/testing-strategy.md`
