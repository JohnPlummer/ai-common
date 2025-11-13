# Docs Folder Organization

Standard structure for documentation folders that complement `.ai/` coding standards, applicable to single-component projects and monorepos.

## Pattern

```markdown
# Single-component project
docs/
├── architecture.md
├── api.md
├── configuration.md
└── deployment.md

# Monorepo or microservices
component-name/docs/
├── architecture.md
├── api.md
└── configuration.md

root/docs/
├── overview.md
├── development-setup.md
└── deployment.md
```

Separation: `.ai/` for HOW to code, `docs/` for WHAT the system does.

## Why Use This Structure

- **Context proximity**: Documentation lives with what it describes
- **Clear separation**: Coding patterns in `.ai/`, system behavior in `docs/`
- **Dual audience**: Works for humans and AI agents understanding system context
- **No duplication**: Standards in `.ai/`, architecture and guides in `docs/`
- **Scalable**: Works for single projects and multi-component systems
- **Portable**: Same pattern across project types

## Content by Folder

### Component docs/ (or root docs/ for single-component)

**Architecture and design**:

- System architecture and component interactions
- Data flow diagrams
- Design decisions

**Reference material**:

- API endpoints and contracts
- Database schema
- Configuration options
- Error codes

**Operational guides**:

- Deployment procedures
- Troubleshooting
- Monitoring and observability

### Root docs/ (monorepo only)

**Cross-cutting concerns**:

- Project overview and purpose
- Monorepo structure
- Development environment setup
- Component interaction patterns

## Guidelines

1. **Keep context close**: Component docs live in component folder
2. **Root for cross-cutting**: Project overview, setup, cross-component patterns
3. **System behavior in docs/**: Architecture, API contracts, data models
4. **Code patterns in .ai/**: Error handling, testing patterns, language standards
5. **No duplication**: Never duplicate `.ai/` standards in `docs/`
6. **Examples in .ai/examples/**: Detailed code examples with context

## Decision Criteria

**Put in docs/** if answering:

- What does this system do?
- How does the architecture work?
- What are the API endpoints?
- What's the database schema?
- How do I deploy this?
- How do components interact?

**Put in .ai/** if answering:

- How should I write error handling?
- What testing patterns do we use?
- How do I structure code layers?
- What naming conventions do we follow?
- What constructor pattern should I use?

## Project Types

### Single-Component Project

```
project/
├── .ai/                    # Coding standards
├── docs/                   # All system documentation
│   ├── architecture.md
│   ├── api.md
│   └── deployment.md
└── README.md              # Quick start, links to docs/
```

### Monorepo

```
monorepo/
├── .ai/                    # Coding standards (shared)
├── docs/                   # Cross-cutting docs only
│   ├── overview.md
│   └── development-setup.md
├── backend/
│   ├── docs/              # Backend-specific docs
│   └── README.md
├── frontend/
│   ├── docs/              # Frontend-specific docs
│   └── README.md
└── README.md              # Project overview
```

### Microservices

```
service-repo/
├── .ai/                    # Service coding standards
├── docs/                   # Service documentation
│   ├── architecture.md
│   ├── api.md
│   └── deployment.md
└── README.md              # Service overview
```

## Anti-patterns

```markdown
Bad: language-standards.md in docs/
Why: Coding standards belong in .ai/

Bad: testing-strategy.md in docs/
Why: Testing patterns belong in .ai/

Bad: Duplicating architecture in root and component docs/
Why: Component architecture in component docs/, system overview in root

Good: docs/architecture.md (single-component)
Why: System architecture lives in docs/

Good: backend/docs/architecture.md (monorepo)
Why: Component architecture lives with component

Good: docs/overview.md (monorepo root)
Why: Cross-component context at root
```

## Related Patterns

- README structure: `.ai/common-standards/readme-documentation.md`
- Component READMEs: `.ai/common-standards/component-readme-pattern.md`
- Documentation organization: `.ai/common-standards/documentation-organization.md`
