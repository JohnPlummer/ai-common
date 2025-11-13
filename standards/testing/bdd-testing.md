# Ginkgo BDD Test Structure

Project-wide BDD testing pattern using Ginkgo v2 and Gomega for all Go tests. Provides expressive, behavior-focused test organization.

## Pattern

```go
import (
    . "github.com/onsi/ginkgo/v2"
    . "github.com/onsi/gomega"
)

var _ = Describe("ComponentName", func() {
    Context("when condition exists", func() {
        It("should behave as expected", func() {
            // Arrange, Act, Assert
            result := performAction()
            Expect(result).To(Equal(expected))
        })
    })
})
```

## Why Use This Pattern

- **Readable**: Tests read like specifications
- **Organized**: Describe/Context/It provides clear hierarchy
- **Expressive**: Gomega matchers are human-readable
- **Consistent**: Same pattern across entire Go codebase
- **Powerful**: Parallel execution, focused specs, rich matchers

## Development Methodology

### Outside-In Development with BDD

- Start with user-facing acceptance tests (Gherkin scenarios when applicable)
- Implement from the outside in: UI → API → Service → Data
- Write tests first, then minimal code to pass
- Refactor only after tests pass

### Contract-First Development

- **Frontend**: Define component APIs and TypeScript interfaces first
- **Backend**: OpenAPI specification drives all API development
- **Integration**: Generated types ensure alignment

## Service Test

```go
// backend/pkg/api/services/location_service_test.go:30
var _ = Describe("LocationService", func() {
    var (
        mockRepo *mocks.MockLocationRepository
        service  services.LocationService
    )

    BeforeEach(func() {
        mockRepo = mocks.NewMockLocationRepository(t)
        service = services.NewLocationService(mockRepo)
    })

    Describe("GetLocations", func() {
        Context("when getting all active locations", func() {
            It("should delegate to repository with activeOnly=true", func() {
                expectedLocations := []types.Location{
                    {ID: "1", Name: "Brighton"},
                }

                mockRepo.EXPECT().
                    GetLocations(mock.Anything, true).
                    Return(expectedLocations, nil)

                locations, err := service.GetLocations(ctx, true)

                Expect(err).NotTo(HaveOccurred())
                Expect(locations).To(HaveLen(1))
            })
        })
    })
})
```

## Repository Integration Test

```go
// backend/pkg/api/repositories/repository_integration_test.go:85
var _ = Describe("ActivityRepository", func() {
    var (
        repo   repositories.ActivityRepository
        testDB *containers.PostgreSQLContainer
    )

    BeforeEach(func() {
        testDB = setupTestDatabase()
        repo = repositories.NewActivityRepository(testDB.Pool)
    })

    It("should insert and retrieve location", func() {
        location := &types.Location{ID: "brighton-uk", Name: "Brighton"}

        err := repo.CreateLocation(ctx, location)
        Expect(err).NotTo(HaveOccurred())

        retrieved, err := repo.GetLocationByID(ctx, "brighton-uk")
        Expect(err).NotTo(HaveOccurred())
        Expect(retrieved.Name).To(Equal("Brighton"))
    })
})
```

## Test Structure

```go
// Describe: Component under test
var _ = Describe("UserService", func() {

    // Context: Scenario or state
    Context("when user exists", func() {

        // It: Expected behavior
        It("should return user details", func() {
            // Test implementation
        })
    })
})
```

## Setup and Teardown

```go
var _ = Describe("Component", func() {
    var (
        component *Component
        mockDep   *MockDependency
    )

    BeforeEach(func() {
        mockDep = NewMockDependency()
        component = NewComponent(mockDep)
    })

    AfterEach(func() {
        component.Cleanup()
    })
})
```

## Gomega Matchers

```go
// Equality
Expect(value).To(Equal(expected))
Expect(value).NotTo(Equal(unexpected))

// Nil checking
Expect(err).NotTo(HaveOccurred())
Expect(ptr).To(BeNil())

// Collections
Expect(slice).To(HaveLen(3))
Expect(slice).To(ContainElement("item"))
Expect(slice).To(BeEmpty())

// Numeric comparisons
Expect(count).To(BeNumerically(">", 10))

// Strings
Expect(str).To(ContainSubstring("partial"))
Expect(str).To(MatchRegexp(`\d{3}`))

// Booleans
Expect(condition).To(BeTrue())
```

## Running Tests

```bash
# Always use make commands, not direct ginkgo
make test                # All tests
make test-unit           # Unit tests only
make test-integration    # Integration tests
make test-race           # With race detection
```

## Guidelines

1. **Use Describe/Context/It**: Maintain clear hierarchy
2. **Name descriptively**: Spec names should be readable sentences
3. **One assertion focus**: Each It block tests one behavior
4. **Setup in BeforeEach**: Initialize fresh state for each test
5. **Limit nesting**: 2-3 levels maximum

## Table-Driven Tests

```go
DescribeTable("validates input",
    func(input string, expected bool) {
        result := validator.Validate(input)
        Expect(result).To(Equal(expected))
    },
    Entry("valid email", "user@example.com", true),
    Entry("invalid email", "not-an-email", false),
)
```

## Related Patterns

- Testcontainers: See `.ai/project-standards/testcontainers.md`
- Mocking: See `.ai/project-standards/mocking.md`

## References

- Ginkgo: <https://onsi.github.io/ginkgo/>
- Gomega: <https://onsi.github.io/gomega/>
- Project testing: `docs/testing-strategy.md`
