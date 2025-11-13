# Go Type and Interface Organization

## Decision

- **Interfaces**: Define where consumed, not where implemented (e.g., RelevanceScorer in `pkg/workers/scorer/`, not `pkg/llm/`)
- **Shared interfaces**: Place in `interfaces.go` if multiple consumers need them within the same package
- **Shared types**: Place in `types.go` when used across multiple files in the package
- **Internal types**: Keep in implementation files where they're used
- **External dependencies**: Wrap with adapters, define our own interfaces for what we need
- **Mocks**: Generate with mockery, store in `tests/mocks/`

