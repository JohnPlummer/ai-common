# AI Common Standards - Progressive Loading Map

Portable, project-agnostic development standards for AI assistants.

**Purpose**: This file provides a progressive loading map for common standards that apply across all Go, TypeScript, and React projects.

**Integration**: Reference this file from your project's `.ai/llms.md` via symlink at `.ai/common/common-llms.md`

## How to Use

Load standards on-demand based on the task at hand. Standards are organized by:

- **Language** (Go, TypeScript)
- **Domain** (Testing, Architecture, Database, Infrastructure, Documentation)

## Go Standards

### Core Go Patterns

**When**: Writing any Go code

- `standards/go/constructors.md` - New* constructor functions
- `standards/go/type-organization.md` - Interface and type placement
- `standards/go/error-wrapping.md` - Error wrapping with fmt.Errorf (%w verb)
- `standards/go/functional-options.md` - Functional options for configuration

### jp-go-* Package Usage

**When**: Using extracted shared packages

- `standards/go/jp-go-errors.md` - Standardized error types, sentinels, HTTP status codes
- `standards/go/jp-go-config.md` - Configuration structs (DatabaseConfig, RedisConfig, HTTPConfig)
- `standards/go/jp-go-pgx-utils.md` - PostgreSQL connection pooling and utilities

## TypeScript/React Standards

**When**: Writing TypeScript or React code

- `standards/typescript/typescript-components.md` - Arrow function components
- `standards/typescript/typescript-interfaces.md` - Interface definitions for props and types
- `standards/typescript/react-hooks.md` - useState, useEffect patterns
- `standards/typescript/custom-hooks.md` - Custom hooks for data fetching and reusable logic
- `standards/typescript/error-boundaries.md` - Error boundary class components
- `standards/typescript/render-props.md` - Render props pattern with generic types
- `standards/typescript/testing-patterns.md` - Vitest and React Testing Library

## Testing Standards

**When**: Writing or modifying tests

### All Testing

- `standards/testing/test-categories.md` - Test types, execution timing, coverage requirements
- `standards/testing/test-timing-patterns.md` - Eventually() vs sleep, avoiding race conditions
- `standards/testing/test-timeout-constants.md` - Standardized timeout constants

### Go Testing (Ginkgo/Gomega)

- `standards/testing/bdd-testing.md` - Ginkgo BDD test structure
- `standards/testing/testcontainers.md` - PostgreSQL container integration
- `standards/testing/mocking.md` - Mockery v3 mock generation
- `standards/testing/test-suite-logging.md` - Suppress INFO/WARN logs in test suites
- `standards/testing/expected-error-logs.md` - Validate and suppress expected error logs

### Error Handling in Tests

- `standards/testing/error-classification.md` - Type-safe error checking with errors.Is
- `standards/testing/error-responses.md` - Standard API error response formats

## Architecture Standards

**When**: Designing or implementing architectural patterns

- `standards/architecture/clean-architecture.md` - Service/repository/handler layers
- `standards/architecture/repository-pattern.md` - Database access pattern

## Database Standards

**When**: Working with databases

- `standards/database/database-transactions.md` - Transaction rollback, timeouts, batch validation
- `standards/go/jp-go-pgx-utils.md` - PostgreSQL connection and helpers

## Infrastructure Standards

**When**: Implementing reliability patterns

- `standards/infrastructure/circuit-breaker.md` - Circuit breaker pattern with sony/gobreaker
- `standards/infrastructure/retry-logic.md` - Retry pattern with exponential backoff and jitter
- `standards/infrastructure/http-middleware.md` - HTTP middleware patterns
- `standards/infrastructure/performance-monitoring.md` - Prometheus metrics for performance validation

## Documentation Standards

**When**: Writing documentation or knowledge capture

### Documentation Organization

- `standards/documentation/documentation-organization.md` - Documentation system architecture and progressive loading
- `standards/documentation/docs-folder-organization.md` - Structure for docs/ folders

### Writing Standards

- `standards/documentation/pattern-documentation.md` - Standard structure for pattern files
- `standards/documentation/markdown-conventions.md` - Markdown formatting conventions
- `standards/documentation/token-efficient-writing.md` - Writing for token efficiency

### Code Documentation

- `standards/documentation/code-references.md` - Referencing real code in documentation
- `standards/documentation/readme-documentation.md` - README structure and content guidelines
- `standards/documentation/component-readme-pattern.md` - Component README pattern

### Knowledge Management

- `standards/documentation/task-decomposition.md` - Task sizing and decomposition criteria
- `standards/documentation/knowledge-capture.md` - Four-tier knowledge capture system

## Loading Strategy by Task Type

| Task Type | Load These Standards |
|-----------|---------------------|
| Go API endpoint | clean-architecture, repository-pattern, bdd-testing, jp-go-errors, jp-go-pgx-utils |
| Database repository | repository-pattern, database-transactions, testcontainers, jp-go-pgx-utils, jp-go-errors |
| Go worker | constructors, functional-options, type-organization, jp-go-errors, jp-go-config |
| Error handling | error-wrapping, jp-go-errors, error-classification, error-responses |
| React component | typescript-components, react-hooks, typescript-interfaces, custom-hooks |
| React error handling | error-boundaries, custom-hooks |
| Writing tests (Go) | bdd-testing, testcontainers, mocking, test-categories, test-timing-patterns |
| Writing tests (React) | testing-patterns |
| Circuit breaker | circuit-breaker, retry-logic |
| HTTP middleware | http-middleware, jp-go-errors |
| Database transactions | database-transactions, jp-go-pgx-utils |
| Configuration | jp-go-config, functional-options |
| Writing documentation | documentation-organization, pattern-documentation, markdown-conventions |
| Writing READMEs | readme-documentation, component-readme-pattern, markdown-conventions |

## Progressive Loading Example

**Example**: Implementing a new Go repository with PostgreSQL

1. **Start**: Load project's `.ai/llms.md` (which loads this file)
2. **Architecture**: Load `standards/architecture/repository-pattern.md`
3. **Database**: Load `standards/go/jp-go-pgx-utils.md`
4. **Transactions**: Load `standards/database/database-transactions.md`
5. **Errors**: Load `standards/go/jp-go-errors.md`
6. **Testing**: Load `standards/testing/bdd-testing.md` and `standards/testing/testcontainers.md`
7. **Constructor**: Load `standards/go/constructors.md` and `standards/go/functional-options.md`

## File Organization

```
ai-common/
├── README.md                           # Integration guide for projects
├── common-llms.md                      # This file (loading map)
├── standards/
│   ├── go/                             # Go language patterns
│   │   ├── constructors.md
│   │   ├── error-wrapping.md
│   │   ├── functional-options.md
│   │   ├── type-organization.md
│   │   ├── jp-go-errors.md
│   │   ├── jp-go-config.md
│   │   └── jp-go-pgx-utils.md
│   ├── typescript/                     # TypeScript/React patterns
│   │   ├── typescript-components.md
│   │   ├── typescript-interfaces.md
│   │   ├── react-hooks.md
│   │   ├── custom-hooks.md
│   │   ├── error-boundaries.md
│   │   ├── render-props.md
│   │   └── testing-patterns.md
│   ├── testing/                        # Testing patterns
│   │   ├── bdd-testing.md
│   │   ├── testcontainers.md
│   │   ├── mocking.md
│   │   ├── test-categories.md
│   │   ├── test-timing-patterns.md
│   │   ├── test-timeout-constants.md
│   │   ├── test-suite-logging.md
│   │   ├── expected-error-logs.md
│   │   ├── error-classification.md
│   │   └── error-responses.md
│   ├── architecture/                   # Architecture patterns
│   │   ├── clean-architecture.md
│   │   └── repository-pattern.md
│   ├── database/                       # Database patterns
│   │   └── database-transactions.md
│   ├── infrastructure/                 # Infrastructure patterns
│   │   ├── circuit-breaker.md
│   │   ├── retry-logic.md
│   │   ├── http-middleware.md
│   │   └── performance-monitoring.md
│   └── documentation/                  # Documentation patterns
│       ├── documentation-organization.md
│       ├── docs-folder-organization.md
│       ├── pattern-documentation.md
│       ├── markdown-conventions.md
│       ├── token-efficient-writing.md
│       ├── code-references.md
│       ├── readme-documentation.md
│       ├── component-readme-pattern.md
│       ├── task-decomposition.md
│       └── knowledge-capture.md
├── examples/                           # Shared code examples
│   └── readme-examples.md
└── prompts/                            # Reusable prompts
    ├── convert-to-common-standard.md
    ├── document-new-standard.md
    ├── extract-patterns.md
    └── generate-docs.md
```

## Contributing New Standards

Use `prompts/convert-to-common-standard.md` to analyze project-specific standards and extract common patterns.

See `README.md` for integration instructions and contribution guidelines.
