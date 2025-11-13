# Component README Pattern

Structure and content guidelines for component READMEs within monorepo or microservice architectures.

## Pattern

Component READMEs differ from project READMEs by focusing on integration context over isolated functionality.

```markdown
# Component Name

> One-sentence description of this component's role in the system

## What & Why (System Context)

[1-2 paragraphs:]
- What role does this component play in the larger system?
- Who consumes this component? (other services, frontend, external systems)
- What problem does it solve within this context?

## Integration Points

[List key interfaces with other components:]
- **Consumed by**: Services/components that depend on this
- **Depends on**: External services, databases, APIs this component uses
- **Data flow**: How data moves through this component

## Quick Start

[Setup commands specific to this component]

## Architecture

[Component-specific architecture, tech stack placement here]

**Tech Stack**: [List stack here, not in title/subtitle]

## Documentation

[Links to .ai/ standards relevant to this component]
```

## Why Use This Pattern

- **Clear scope**: Readers understand component boundaries within system
- **Integration-focused**: Emphasizes how component fits in larger architecture
- **Consumer-oriented**: Identifies who depends on this component
- **Appropriate context**: Includes necessary parent context without duplicating entire project README
- **Tech stack placement**: Stack details in Architecture section, not title/subtitle

## Examples from Codebase

### Pipeline Package (Good Integration Context)

```markdown
# Configuration Package

> Provides configuration loading and validation for the pipeline service using Viper

## What & Why

This package centralizes configuration management for the pipeline service, supporting
multiple sources (files, environment variables) with validation. Other pipeline packages
consume this to access database credentials, API keys, and worker settings.

## Integration Points

- **Consumed by**: All pipeline workers, database package, platform integrations
- **Depends on**: Environment variables, config.yaml files
- **Data flow**: Loads → Validates → Provides typed config to consumers

## Architecture

**Tech Stack**: Go, Viper, validator/v10
```

### E2E Tests (Good Consumer Identification)

```markdown
# System-Wide End-to-End Tests

> Validates complete system functionality across all services (Frontend → Backend → Pipeline)

## What & Why

These tests verify critical user journeys spanning multiple services. Unlike unit or
integration tests in individual components, these validate the entire system works together.

## Integration Points

- **Tests**: Frontend → Backend API → Pipeline workers → Database
- **Requires**: All services running, test database, mocked external APIs
- **Validates**: User journeys, cross-service data flow, API contracts
```

## Guidelines

1. **System context first**: Explain component's role before implementation details
2. **Identify consumers**: Be explicit about who/what depends on this component
3. **Integration over isolation**: Focus on how this fits in the system, not just what it does
4. **Minimal parent duplication**: Reference parent project context, don't copy entire overview
5. **Tech stack in Architecture**: Never in title or subtitle

## Anti-Patterns

```markdown
# Bad: Tech stack as subtitle
> React 18 + TypeScript frontend component library

# Good: Role in system as subtitle
> Reusable UI components consumed by all frontend features

---

# Bad: No integration context
This package handles Reddit API integration.

# Good: Clear integration context
This package provides Reddit data to the pipeline ingestion workers,
abstracting the third-party reddit-client library and providing
mockable interfaces for testing.

---

# Bad: Isolated "what it does"
Features:
- Load configuration files
- Parse environment variables
- Validate settings

# Good: System-oriented "who needs this"
This package centralizes configuration for all pipeline workers.
Workers import this to access database credentials, API keys,
and worker pool settings without directly parsing env vars.
```

## Related Patterns

- `.ai/common-standards/readme-documentation.md` - Parent README standard
- `.ai/common-standards/documentation-organization.md` - Documentation system
- `.ai/common-standards/markdown-conventions.md` - Markdown formatting
