# Clean Architecture Pattern

Service/repository/handler layer separation following Clean Architecture principles. Clear boundaries between business logic and infrastructure.

## Pattern

```go
// Handler layer (HTTP/API)
type Handler struct {
    service Service
}

// Service layer (business logic)
type Service interface {
    DoSomething(ctx context.Context, input Input) (Output, error)
}

// Repository layer (data access)
type Repository interface {
    Save(ctx context.Context, entity Entity) error
    Get(ctx context.Context, id string) (*Entity, error)
}
```

## Why Use This Pattern

- **Testable**: Mock dependencies at each layer
- **Maintainable**: Clear separation of concerns
- **Flexible**: Swap implementations without changing business logic
- **Scalable**: Easy to add new features

## Layer Responsibilities

### Handler Layer

- HTTP request/response handling
- Input validation
- Error code mapping (500, 404, 400)
- Delegates to service layer

```go
// backend/pkg/api/handlers/activity_handler.go
func (h *ActivityHandler) GetActivities(w http.ResponseWriter, r *http.Request) {
    locationID := chi.URLParam(r, "locationId")

    activities, err := h.service.GetActivities(r.Context(), locationID, nil, nil)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(activities)
}
```

### Service Layer

- Business logic and rules
- Orchestrates repository calls
- Transaction coordination

```go
// backend/pkg/api/services/location_service.go:67
type LocationService interface {
    GetLocations(ctx context.Context, activeOnly *bool) ([]types.Location, error)
}

type locationService struct {
    repository LocationRepository
}

func (s *locationService) GetLocations(ctx context.Context, activeOnly *bool) ([]types.Location, error) {
    if activeOnly == nil {
        defaultActive := true
        activeOnly = &defaultActive
    }

    locations, err := s.repository.GetLocations(ctx, *activeOnly)
    if err != nil {
        return nil, fmt.Errorf("failed to get locations: %w", err)
    }

    return locations, nil
}
```

### Repository Layer

- Database queries
- Data mapping (DB ↔ domain models)
- Transaction execution
- No business logic

```go
// backend/pkg/api/repositories/postgres_activity_repository.go
func (r *postgresActivityRepository) GetAll(ctx context.Context) ([]Activity, error) {
    query := `SELECT id, title, description FROM activities`

    rows, err := r.pool.Query(ctx, query)
    if err != nil {
        return nil, fmt.Errorf("failed to query activities: %w", err)
    }
    defer rows.Close()

    var activities []Activity
    for rows.Next() {
        var a Activity
        if err := rows.Scan(&a.ID, &a.Title, &a.Description); err != nil {
            return nil, fmt.Errorf("failed to scan: %w", err)
        }
        activities = append(activities, a)
    }

    return activities, nil
}
```

## Interface Definitions

Define interfaces in the layer that *uses* them, not where they're implemented:

```go
// In service package - service defines what it needs
package services

type LocationRepository interface {
    GetLocations(ctx context.Context, activeOnly bool) ([]types.Location, error)
}

type locationService struct {
    repository LocationRepository  // Depends on interface
}

// In repository package - implements interface
package repositories

var _ services.LocationRepository = (*postgresLocationRepository)(nil)
```

## Dependency Flow

```
Handler → Service → Repository → Database
         (interface)  (interface)
```

Dependencies point inward:

- Handler depends on Service interface
- Service depends on Repository interface
- Repository implements interface (no dependencies on service)

## Testing Each Layer

### Handler Tests

```go
It("should return activities for location", func() {
    req := httptest.NewRequest("GET", "/locations/brighton-uk/activities", nil)
    w := httptest.NewRecorder()

    handler.GetActivities(w, req)

    Expect(w.Code).To(Equal(200))
})
```

### Service Tests (with Mocks)

```go
It("should get locations from repository", func() {
    mockRepo.EXPECT().
        GetLocations(mock.Anything, true).
        Return(expectedLocations, nil)

    locations, err := service.GetLocations(ctx, true)

    Expect(err).NotTo(HaveOccurred())
    Expect(locations).To(HaveLen(2))
})
```

### Repository Tests (with Database)

```go
It("should persist and retrieve activities", func() {
    err := repo.Create(ctx, activity)
    Expect(err).NotTo(HaveOccurred())

    retrieved, err := repo.GetByID(ctx, activity.ID)
    Expect(err).NotTo(HaveOccurred())
    Expect(retrieved.Title).To(Equal(activity.Title))
})
```

## Guidelines

1. **Depend on interfaces**: Not concrete types
2. **Define interfaces where used**: Service defines Repository interface
3. **Keep layers focused**: Each layer has single responsibility
4. **Pass context**: Always accept context as first parameter
5. **No skip layers**: Handler → Service → Repository (no shortcuts)

## Common Mistakes

```go
// Bad: Handler directly accesses repository
handler := &Handler{repo: repo}  // Skips service layer

// Good: Handler uses service
handler := &Handler{service: service}

// Bad: Service depends on concrete type
type Service struct {
    repo *PostgresRepository  // Tightly coupled
}

// Good: Service depends on interface
type Service struct {
    repo Repository  // Flexible, testable
}
```

## Related Patterns

- Repository Pattern: See `.ai/project-standards/repository-pattern.md`
- Mocking: See `.ai/project-standards/mocking.md`

## References

- Go interfaces: <https://go.dev/tour/methods/9>
- Project overview: `docs/service-architecture.md`
