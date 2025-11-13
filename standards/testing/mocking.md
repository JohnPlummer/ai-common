# Mockery v3 Mocking Pattern

Project uses Mockery v3 for auto-generated mocks with both traditional On() and modern EXPECT() patterns.

## Pattern

```go
import "github.com/JohnPlummer/some-things-to-do/pipeline/tests/mocks"

// Create mock
mockRepo := mocks.NewMockRepository(t)

// Traditional On() pattern
mockRepo.On("Get", mock.Anything, "123").
    Return(&Entity{}, nil)

// Modern EXPECT() pattern (type-safe)
mockRepo.EXPECT().Get(mock.Anything, "123").
    Return(&Entity{}, nil)
```

## Why Use This Pattern

- **Auto-generated**: Mockery generates mocks from interfaces
- **Type-safe**: EXPECT() provides compile-time checking
- **Consistent**: Standard approach across codebase
- **Maintainable**: Mocks update with interface changes
- **Flexible**: Both On() and EXPECT() patterns supported

## Configuration

**Location**: `.mockery.yaml` in service root

```yaml
all: false
structname: "Mock{{.InterfaceName}}"
template: testify
filename: "{{.InterfaceName}}_mock.go"
pkgname: mocks
dir: ./tests/mocks
packages:
  github.com/JohnPlummer/some-things-to-do/pipeline/pkg/repository:
    interfaces:
      PostRepository:
      ActivityRepository:
```

## Generating Mocks

```bash
# Generate all configured mocks
make mocks

# Regenerate (clean + generate)
make regenerate-mocks

# Verify mocks are up-to-date (CI)
make verify-mocks
```

## Traditional On() Pattern

```go
mockRepo := mocks.NewMockPostRepository(t)

// Setup expectation
mockRepo.On("GetPostsForExtraction", mock.Anything, 10).
    Return([]*Post{}, nil)

// Use mock
result, err := extractor.Process(mockRepo)

// Verify
mockRepo.AssertExpectations(t)
```

**Pros**: Familiar, widely used, simple

**Cons**: String-based method names, no IDE autocomplete

## Modern EXPECT() Pattern

```go
mockRepo := mocks.NewMockPostRepository(t)

// Setup expectation (type-safe)
mockRepo.EXPECT().
    GetPostsForExtraction(mock.Anything, 10).
    Return([]*Post{}, nil)

// Use mock
result, err := extractor.Process(mockRepo)

// Verify
mockRepo.AssertExpectations(t)
```

**Pros**: Type-safe, IDE autocomplete, refactor-friendly

**Cons**: Newer pattern, fewer examples

## Advanced Patterns

### RunAndReturn (Dynamic Behavior)

```go
mockScorer.EXPECT().
    ScorePost(mock.Anything, mock.Anything).
    RunAndReturn(func(ctx context.Context, post *Post) (*ScoreResponse, error) {
        // Dynamic logic based on actual arguments
        if post.Score > 70 {
            return &ScoreResponse{Score: 95}, nil
        }
        return &ScoreResponse{Score: 50}, nil
    })
```

### Call Count Constraints

```go
// Exactly once
mockRepo.EXPECT().Save(mock.Anything, mock.Anything).Once()

// Twice
mockRepo.EXPECT().Save(mock.Anything, mock.Anything).Twice()

// N times
mockRepo.EXPECT().Save(mock.Anything, mock.Anything).Times(3)
```

### Custom Matchers

```go
mockRepo.EXPECT().
    SavePost(mock.MatchedBy(func(p *Post) bool {
        return p.Score > 70
    })).
    Return(nil)
```

## Manual Mocks for Private Interfaces

Use manual mocks for trivial private interfaces:

```go
// Private testing seam interface
type batchProcessor interface {
    ProcessBatch(ctx context.Context) (*BatchStatistics, error)
}

// Manual mock (simple, test-only)
type mockBatchProcessor struct {
    processFunc func(ctx context.Context) (*BatchStatistics, error)
    callCount   int
}

func (m *mockBatchProcessor) ProcessBatch(ctx context.Context) (*BatchStatistics, error) {
    m.callCount++
    if m.processFunc != nil {
        return m.processFunc(ctx)
    }
    return &BatchStatistics{Total: 10, Processed: 10}, nil
}
```

**When to use manual mocks:**

- Interface is private (lowercase)
- Single method or trivial (2-3 methods)
- Only used for testing in same package

**When to use generated mocks:**

- Interface is public (uppercase)
- Part of public API
- Multiple methods or complex
- Interface may evolve

## Guidelines

1. **Use EXPECT() for new tests**: Type safety and IDE support
2. **Use On() for existing tests**: Maintain consistency
3. **Manual mocks for private interfaces**: Simple, focused testing
4. **Generated mocks for public interfaces**: Listed in .mockery.yaml
5. **Always use mock.Anything for context**: Flexible context types
6. **Verify expectations**: Call AssertExpectations(t)

## Adding New Mocks

1. Add interface to `.mockery.yaml`:

```yaml
packages:
  github.com/YourProject/pkg/newpackage:
    interfaces:
      NewInterface:
```

2. Generate: `make mocks`
3. Import: `import "github.com/YourProject/tests/mocks"`
4. Use: `mockNew := mocks.NewMockNewInterface(t)`

## Related Patterns

- BDD Testing: See `.ai/project-standards/bdd-testing.md`
- Clean Architecture: See `.ai/project-standards/clean-architecture.md`

## References

- Mockery v3: <https://vektra.github.io/mockery/>
- Implementation: `pipeline/.mockery.yaml`
- Guide: `pipeline/docs/mocking-guide.md`
