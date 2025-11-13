# Project Standards

> Project-specific patterns and architecture decisions for the Some Things To Do codebase.

## What & Why

These patterns document how we implement common concerns in this specific codebase. While common-standards/ contains portable patterns usable in any Go/React project (error wrapping, constructors, hooks), project-standards/ contains our architectural decisions and implementation approaches.

**Use project-standards when**: You need to understand how we structure backend services (clean architecture), run tests (Ginkgo BDD), manage database access (repository pattern), or integrate with specific tools (testcontainers).

**Use common-standards when**: You need general patterns like error wrapping, functional options, or TypeScript component structure that apply across projects.

The distinction: common-standards are "how to write good Go/React code", project-standards are "how we write code in this codebase".

## Quick Start

### Essential Commands

```bash
# Development
make dev                        # Start frontend + backend
make dev-frontend               # Frontend only (localhost:5173)
make dev-backend                # Backend only (localhost:8080)

# Testing & Quality
make check                      # ALWAYS run before committing
make test                       # All tests
make test-unit                  # Unit tests only
make test-integration           # Integration tests (needs Docker)
make test-race                  # Run tests with race detector

# Code Generation
make codegen                    # Generate types from OpenAPI spec

# Database
make db-setup                   # Initial database setup
make migrate-up                 # Apply migrations
make migrate-status             # Check migration status
```

## Architecture

- **Clean Architecture**: Backend follows service/repository/handler layers
- **BDD Testing**: Ginkgo/Gomega for all Go tests (NEVER use standard testing package)
- **Contract-First**: OpenAPI 3.0 drives all API development
- **Test Containers**: PostgreSQL containers for integration tests

### OpenAPI-First Development

- **Always** update `shared/api/openapi.yaml` first
- Run `make codegen` after API changes
- Generated types in `backend/pkg/api/types/` and `frontend/src/types/`

## Project Structure

```
frontend/           # React app (Vite, TypeScript, MUI)
backend/            # Go API (Chi router, Clean Architecture)
pipeline/           # Data processing (Reddit, OpenAI integration)
database/migrations/# Shared PostgreSQL migrations
shared/api/         # OpenAPI spec (source of truth)
.ai/                # AI assistant documentation
```

## Technology Stack

### Frontend

- **Core**: React 18+, TypeScript (strict mode), Vite
- **UI**: Material-UI (MUI) component library
- **Testing**: Vitest, React Testing Library, jest-cucumber
- **Quality**: ESLint, Prettier, TypeScript strict

### Backend

- **Core**: Go, PostgreSQL, OpenAPI-driven
- **Framework**: oapi-codegen for contract-first development
- **Testing**: Ginkgo + Gomega for BDD
- **Quality**: go fmt, go vet, staticcheck

### Pipeline

- **Workers**: Go workers for data processing
- **Integrations**: Reddit API, OpenAI API

## Environment Setup

Required environment variables:

```bash
# Required
DATABASE_URL=postgres://somethings:password@localhost:5432/somethingstodo?sslmode=disable
API_URL=http://localhost:8080
VITE_API_URL=http://localhost:8080      # Same as API_URL in dev

# For E2E tests & pipeline
REDDIT_CLIENT_ID, REDDIT_CLIENT_SECRET, OPENAI_API_KEY
```

## Critical Context

### Recent Migrations

- **ST-212**: Migrated to testcontainers PostgreSQL module for all database tests
- **ST-233**: Migrated all Go tests from standard testing to Ginkgo/Gomega
- **ST-246**: Consolidated database migrations to `database/migrations/`

### Common Issues

1. **Test failures**: Ensure Docker is running for integration tests (required for testcontainers)
2. **Type generation**: Always run `make codegen` after OpenAPI changes
3. **Ginkgo not found**: Test file must end with `_test.go` and have suite file
4. **Database test failures**: Check Docker daemon is running and has sufficient resources
5. **Testcontainers timeout**: Increase test timeout or check Docker pull permissions

## Development Workflow

- **Branch naming**: `ST-XXX-description` (with ticket) or `feature/description`
- **Commit format**: `ST-XXX: type(scope): description`
- **Always run** `make check` before committing
- **Reference existing patterns** in codebase

## Available Standards

Load these on-demand based on task context (see `.ai/llms.md` for loading strategy):

**Backend Architecture**:

- Clean architecture pattern (service/repository/handler layers)
- Repository pattern for database access
- Database transaction management with pgx
- HTTP middleware patterns
- Error response formats

**Testing Approaches**:

- Ginkgo BDD test structure (all Go tests)
- Testcontainers for PostgreSQL integration tests
- Mockery v3 mock generation
- E2E test orchestrator pattern

**Configuration & Error Handling**:

- Viper configuration management
- Type-safe error classification (errors.Is)

**Frontend**:

- React Router v7 navigation patterns

## Documentation

- **Progressive loading map**: `.ai/llms.md` - what to load for each task type
- **Portable patterns**: `.ai/common-standards/` - reusable Go/React patterns
- **Project guides**: `docs/` - comprehensive setup and architecture docs
