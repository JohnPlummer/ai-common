# Knowledge Capture

Four-tier system for capturing and organizing project knowledge.

## Pattern

```
Ticket-specific → Jira comments
Repeatable pattern → .ai/standards/*.md
Current context → context.md
Permanent gotcha → memory.md
```

## Why Use This Pattern

- **Single source of truth**: Each piece of knowledge lives in exactly one place
- **Progressive discovery**: Patterns emerge from repeated solutions
- **Searchable history**: Jira provides ticket-scoped search
- **Token efficiency**: Load only relevant context
- **Clear boundaries**: Decision tree prevents duplication

## Where Knowledge Lives

### 1. Jira Comments (Ticket-Specific)

**What**: Plans, decisions, failures for this ticket

**When**: Always document ticket work here

**Duration**: Permanent (lives with ticket)

**Example**: "Tried sliding window rate limiting but conflicts with cache keys. Switched to token bucket."

### 2. .ai/standards/ (Patterns & Practices)

**What**: Repeatable approaches

**When**: Something becomes "the way we do X" (after 2-3 occurrences)

**Duration**: Until pattern evolves

**Example**: Create `.ai/project-standards/rate-limiting.md` after using token bucket 3 times

### 3. context.md (Current Work)

**What**: Active work, recent discoveries, temporary workarounds

**When**: Cross-ticket discoveries during implementation

**Duration**: Clear when work completes or info becomes stale

**Example**: "Redis pool exhausts under load - temporary fix: increased pool size. TODO: investigate proper management"

### 4. memory.md (Project Knowledge)

**What**: Constraints, limits, gotchas (NOT patterns)

**When**: Permanent project-specific knowledge

**Duration**: Rarely changes

**Example**: "Reddit API: 60 req/min limit. OpenAI latency spikes 3pm-5pm PST."

## Decision Tree

After learning something:

```
1. Ticket-specific implementation?
   → Jira comment

2. Repeatable pattern (seen 2-3 times)?
   → .ai/standards/*.md

3. Active work or recent discovery?
   → context.md

4. Permanent constraint or gotcha?
   → memory.md
```

## Pattern Promotion Flow

When something recurs 2-3 times:

1. **Notice**: Same approach in multiple tickets
2. **Extract**: Create/update `.ai/standards/*.md`
3. **Document**: Use pattern structure (Why, When, How, Examples)
4. **Reference**: Add to `.ai/llms.md` loading map
5. **Link**: Reference from original Jira tickets
6. **Clean**: Remove from context.md (now it's a standard)

**Example**:

```
ST-445: Used token bucket rate limiting (Jira comment)
ST-446: Used token bucket again (Jira comment)
ST-450: Used token bucket third time

Action:
- Create .ai/project-standards/rate-limiting.md
- Add to .ai/llms.md loading map
- Link from ST-445, ST-446, ST-450
- Remove from context.md
```

## Lifecycle Rules

### context.md

**Update when**:

- Making cross-ticket discoveries
- Tracking active investigations
- Documenting temporary workarounds

**Clear when**:

- Work completes
- Issue resolved
- Info no longer relevant

### memory.md

**Update when**:

- Discovering permanent constraints
- Documenting recurring gotchas
- Adding integration quirks

**Clear when**:

- Constraint no longer applies
- Gotcha permanently fixed

### .ai/standards/

**Update when**:

- Pattern emerges (2-3 occurrences)
- Pattern evolves or improves
- Better approach discovered

### Jira

**Never clear**: Permanent searchable history

## Anti-patterns

```bash
# Bad: Duplicating info across files
Jira comment + context.md + memory.md (same info)

# Good: One location per piece of knowledge
Jira comment (ticket-specific)

# Bad: Creating pattern from single occurrence
ST-445 used token bucket → Create pattern immediately

# Good: Wait for pattern to emerge
ST-445, ST-446, ST-450 all used token bucket → Create pattern

# Bad: Keeping stale context
context.md has 6-month-old "evaluating alternatives"

# Good: Clear when work completes
Remove from context.md when decision made
```

## Related

- `.ai/project-standards/jira-workflow.md` - Jira-centric planning
- `.ai/common-standards/documentation-organization.md` - Progressive loading
- `.ai/common-standards/pattern-documentation.md` - Pattern structure
