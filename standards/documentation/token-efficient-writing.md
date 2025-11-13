# Token-Efficient Writing

Writing techniques that minimize token usage while maximizing information density for AI-optimized documentation.

## Pattern

```markdown
## Pattern
```go
code example
```

## Why Use This Pattern

- **Benefit**: Concrete explanation
- **Another benefit**: Specific reason

## Examples from Codebase

```go
// file/path.go:42
real code
```

```

60% examples, 40% explanation. Code first, prose second.

## Why Use Token-Efficient Writing

- **Context window limits**: AI assistants have finite token budgets
- **Processing speed**: Fewer tokens mean faster comprehension
- **Focus**: Dense writing forces clarity
- **RAG optimization**: Smaller chunks improve retrieval

## Core Principle: 60/40 Rule

### 60% Examples

```go
// Good: Code shows pattern directly
if err != nil {
    return fmt.Errorf("failed to process: %w", err)
}
```

Code demonstrates faster than prose explains.

### 40% Explanation

```markdown
## Why Use This Pattern
- **Error chains**: Preserves original error
- **Context**: Adds operation details
- **Debugging**: Traces error origin
```

Brief explanations with bold labels for scanning.

## Sentence Structure

### Direct and Active

```markdown
Good: Use fmt.Errorf with %w for error wrapping.
Avoid: You should consider using fmt.Errorf with the %w formatting verb.

Good: Tests use Ginkgo BDD framework.
Avoid: In this project, we have decided to utilize the Ginkgo BDD framework.
```

Remove filler. Use imperative mood. State facts directly.

### Token-Heavy Phrases to Avoid

```markdown
Avoid: "It is important to note that..."  Use: (state the fact)
Avoid: "In order to..."  Use: "To..."
Avoid: "This allows you to..."  Use: "This enables..." or describe capability
Avoid: "One of the benefits..."  Use: "**Benefit**: ..." in bullet list
```

## Code Over Prose

```markdown
Bad (prose-heavy):
"When creating a service in Go, follow the constructor pattern with New prefix."

Good (code-first):
```go
// backend/services/user_service.go:23
func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}
```

Constructor pattern: New* prefix, returns pointer.

```

Let code demonstrate the pattern. Add minimal prose for context.

## List vs Paragraph

```markdown
Good (scannable):
Three benefits:
- Fast error tracing
- Preserves error chain
- Adds operation context

Avoid (paragraph):
There are three main benefits. First, fast error tracing. Second, preserves error chain. Third, adds operation context.
```

Lists are more scannable than paragraphs.

## Bold for Scanning

```markdown
Good: Use **fmt.Errorf** with the **%w** verb for error wrapping.
Bad: **This** is **too** much **bold** and **makes** everything **hard** to **read**.
```

Bold key terms only. Limit to 2-3 bold terms per sentence maximum.

## Reference Concepts, Not Files

When context is loaded, AI knows what files exist:

```markdown
Good: "Load error handling and constructor patterns"
Avoid: "Load error-handling.md, constructors.md"

Good: "See retry patterns"
Avoid: "See .ai/common-standards/retry-patterns.md"
```

Only specify files for code references (backend/services/user.go:42) or bootstrap instructions.

## Guidelines

1. **Examples first**: Lead with code, not explanation
2. **Active voice**: "Use pattern" not "Pattern can be used"
3. **Cut filler**: Remove "it is important", "you should", "in order to"
4. **Bold strategically**: Key terms only
5. **Lists over paragraphs**: More scannable
6. **Document reality**: What IS, not what SHOULD BE
7. **Target 60/40**: 60% examples, 40% explanation
8. **Compress ruthlessly**: Same info in fewer tokens
9. **Reference concepts**: Not file names (when context loaded)

## Token Budget Targets

```markdown
Common standards: 400-600 tokens
Project standards: 400-600 tokens
README files: 200-400 tokens
```

Write to target. Use token counter during authoring.
