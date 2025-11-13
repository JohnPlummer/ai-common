# Test Categories

Project-wide test categorization defining purpose, execution timing, and coverage requirements.

## Pattern

```go
// Unit Test - Fast, isolated
It("should map category correctly", func() {
    result, err := mapper.Map("restaurant")
    Expect(err).NotTo(HaveOccurred())
    Expect(result.Mapped).To(Equal(CategoryFoodDrink))
})

// Integration Test - Real database
It("should save posts to database", func() {
    result, err := postRepo.SavePosts(ctx, posts)
    Expect(err).NotTo(HaveOccurred())
    Expect(result.InsertedPosts).To(Equal(2))
})
```

## Test Categories

**Unit Tests**: Individual functions, no external dependencies

- Every save, pre-commit, all CI
- 80%+ coverage for business logic
- <50ms per test, extensive mocking

**Integration Tests**: Real dependencies (testcontainers)

- All CI runs, local as needed
- Critical interaction paths
- Location: `tests/integration/`

**Feature Tests**: User-facing functionality (BDD)

- CI on all branches
- Core user journeys
- Location: `tests/features/`

**E2E Tests**: Full system validation

- Manual, pre-release only
- Complete data pipelines
- Location: `tests/e2e/`

## Testing Pyramid

60% Unit / 30% Integration / 10% Feature+E2E

## Execution

```bash
make test              # All tests
make test-unit         # Unit only
make test-integration  # Integration (requires Docker)
make check             # Tests + lint + fmt
```

## Timing Assertions Anti-Pattern

**Problem**: Absolute timing thresholds fail in CI due to resource contention.

```go
// DON'T: Flaky in CI
start := time.Now()
result := service.Process(data)
Expect(time.Since(start)).To(BeNumerically("<", 5*time.Millisecond))  // Fails

// DO: Test correctness only
result := service.Process(data)
Expect(result).To(BeValid())
```

**Exception**: Keep minimum timing when testing timeout behavior:

```go
// OK: Verify timeout respected
start := time.Now()
err := op.WaitWithBackoff(ctx, 1)
Expect(time.Since(start)).To(BeNumerically(">=", 100*time.Millisecond))
```

**Solution**: Tests validate correctness. Prometheus tracks performance in production.

## Related

- `.ai/project-standards/bdd-testing.md`
- `.ai/project-standards/performance-monitoring.md`
