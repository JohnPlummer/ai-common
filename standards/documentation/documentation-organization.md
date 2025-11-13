# Documentation Organization

System architecture for organizing AI-optimized documentation using progressive loading.

## Pattern

```
Always-loaded (<500 tokens total):
├── agents.md (~400 tokens) - Behavioral rules, task context pointers
├── CLAUDE.md (~200 tokens) - Entry point, quick commands
└── .ai/llms.md (~800 tokens) - Discovery map, loading strategy

On-demand (400-600 tokens each):
├── .ai/common-standards/*.md - Portable patterns
└── .ai/project-standards/*.md - Project-specific patterns
```

Progressive loading: Minimal in always-loaded, detail in on-demand files.

## Why Use This Organization

- **Context window efficiency**: Load only needed content
- **Faster processing**: Less scanning for relevant info
- **Single source of truth**: No duplication
- **Scalable**: Add standards without bloating always-loaded files

## Always-Loaded Files

### agents.md (~400 tokens)

Behavioral rules and task context pointers. Contains core principles, quality gates, progressive loading task list (pointers only, not detailed instructions).

### CLAUDE.md (~200 tokens)

Entry point with load instructions for .ai/llms.md, quick commands, architecture pointer.

### .ai/llms.md (~800 tokens)

Discovery map with architecture overview, available standards list with one-line descriptions, loading strategy table (task type → standards), file organization tree.

## On-Demand Files

### Common Standards (400-600 tokens each)

Portable patterns usable in any project. Contains pattern definition with code example, why section with benefits, real codebase examples, guidelines and anti-patterns.

### Project Standards (400-600 tokens each)

Project-specific patterns and conventions. Contains architecture patterns unique to this project, project-specific implementations, integration patterns.

## Single Source of Truth

Each concept documented in exactly one place.

**Where Content Belongs:**

- Behavioral rules → `agents.md`
- Discovery/routing → `.ai/llms.md`
- Pattern details → `.ai/common-standards/*.md`
- Project specifics → `.ai/project-standards/*.md`

## Progressive Loading Budget

```
Always-loaded:
agents.md:        ~400 tokens
CLAUDE.md:        ~200 tokens
.ai/llms.md:      ~800 tokens
-----------------
Total:           ~1,400 tokens
```

Load minimal content in every session.

## File Size Targets

```
Always-loaded files:
- agents.md:           400 tokens
- CLAUDE.md:           200 tokens
- .ai/llms.md:         800 tokens

On-demand files:
- common-standards/*:  400-600 tokens
- project-standards/*: 400-600 tokens

Excluded from limits:
- .ai/prompts/*:       No limit (operational templates)
- .ai/examples/*:      No limit (detailed examples)
```

## Organization Anti-Patterns

```markdown
Bad: Adding 80-line detailed section to agents.md
Why: Violates progressive loading, doubles file size

Bad: Documenting same pattern in multiple files
Why: Violates single source of truth

Bad: Listing all files when context loaded
Why: Becomes stale, adds no value, AI knows files
```

## Validation Checklist

- [ ] Always-loaded files under budget (agents.md ~400 tokens)
- [ ] No duplication across files
- [ ] Detailed guidance in on-demand files
- [ ] New standards are 400-600 tokens
- [ ] Each concept has single source of truth
