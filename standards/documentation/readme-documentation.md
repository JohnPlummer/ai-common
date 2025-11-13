# README Documentation

Standard structure and content guidelines for README.md files. READMEs are human-oriented entry points providing quick understanding and setup.

## Purpose: Human vs AI Documentation

**README.md (for humans)**: Problem/solution context, quick start, navigation, setup, links to details
**.ai/ docs (for LLMs)**: Comprehensive patterns, code examples, detailed guidance

**Relationship**: READMEs link to .ai/ docs for implementation details. Minimal overlap accepted (architecture diagrams, test overview), but avoid duplicating patterns.

## Critical Mindset: Explain WHY Before HOW

READMEs fail when they assume the reader already understands the context. A developer discovering your project needs to know:

1. **What problem exists?** (the pain point)
2. **Why does this project exist?** (the motivation)
3. **Who is this for?** (target users)
4. **What does it do?** (the solution)

Only AFTER establishing context should you explain:

5. **How to use it?** (quick start)
6. **How is it built?** (architecture, tech stack)

## Standard Structure

```markdown
# Title

> One-sentence value proposition (focus on WHAT problem this solves, not tech stack)

## What & Why

[2-3 paragraphs explaining:]
- What problem does this solve?
- Who is this for?
- What does this do? (features/capabilities from user perspective)
- Why does this exist? (motivation, unique value)

## Quick Start
[Commands to get running]

## [Context-Specific Sections]
Architecture, Directory Structure, Testing, Development
(Choose sections appropriate for your context)

## Documentation
[Links to .ai/ standards and detailed guides]
```

## Content Guidelines

**Include**:

- Problem and solution context
- Target users and use cases
- Setup and quick start
- High-level architecture (diagrams OK)
- Links to .ai/ docs for detailed patterns

**Lead with context, not implementation**:

- Start with WHAT and WHY (problem, solution, value)
- Then HOW to use (quick start, commands)
- Then technical details (architecture, stack)

**Exclude**:

- Detailed pattern implementations (→ .ai/ docs)
- Tech stack as primary description (→ Architecture section)
- Comprehensive code examples (→ .ai/ docs or example directories)

## Anti-Patterns

**Bad**: Tech stack as subtitle
> "React 18 + TypeScript + Vite frontend with MUI components"

**Good**: Value proposition as subtitle
> "Discover local activities beyond typical tourist recommendations"

**Bad**: Vague purpose
"A Go-based REST API server for discovering activities"

**Good**: Specific problem and solution
"REST API aggregating events from multiple sources, scoring for relevance, surfacing hidden gems typical aggregators miss"

See [.ai/examples/readme-examples.md](../examples/readme-examples.md) for detailed before/after transformations.

## README Types

**Project Root**: Project overview, getting started, navigation (all projects)
**Component/Module**: Component setup, architecture, workflows (monorepo modules, microservices)
**Package/Library**: Package usage, API reference, configuration (reusable packages)
**Specialized**: Specific system usage (test suites, tooling, docs systems)

## Validation Checklist

- [ ] Explains what problem this solves (not just what it does)
- [ ] Identifies target users or use cases
- [ ] Purpose comes BEFORE technical details
- [ ] Links to .ai/ docs instead of duplicating patterns
- [ ] Tech stack in Architecture section, not title/subtitle

## Related Patterns

- `.ai/common-standards/documentation-organization.md` - Documentation system
- `.ai/common-standards/markdown-conventions.md` - Markdown formatting
- `.ai/examples/readme-examples.md` - Detailed before/after examples
