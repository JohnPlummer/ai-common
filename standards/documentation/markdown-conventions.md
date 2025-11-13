# Markdown Formatting Conventions

Standardized markdown formatting for AI-optimized documentation.

## Pattern

```markdown
# Document Title

Brief description in one sentence.

## Major Section

Content with **bold emphasis** for key terms.

### Subsection

Content with code examples:

```language
code here
```

- **Bullet item**: Description
- **Another item**: Description

1. Numbered step one
2. Numbered step two

```

## Why Use These Conventions

- **Consistency**: All documentation follows same structure
- **Scannable**: Clear hierarchy aids AI parsing
- **Token-efficient**: Reduces unnecessary formatting tokens
- **Readable**: Works well in CLI and web renderers
- **Standard**: Follows CommonMark specification

## Heading Hierarchy

### Usage Rules

```markdown
# Title - File title only (once per file)
## Major Section - Primary divisions (3-6 per file)
### Subsection - Secondary divisions (as needed)
#### Detail Level - Rare, use sparingly
```

### Guidelines

1. **One H1 per file**: Document title only
2. **Logical structure**: H2 for main topics, H3 for subtopics
3. **No skipping levels**: Don't jump from H2 to H4
4. **Descriptive headings**: State content clearly

### Example from Codebase

```markdown
// error-wrapping.md:1
# Error Wrapping with fmt.Errorf

## Pattern

## Why Use This Pattern

## Examples from Codebase

### Repository Error Wrapping

### Worker Error Wrapping
```

## Code Blocks

### Syntax Highlighting

Always specify language for syntax highlighting:

```markdown
```go
func example() error {
    return nil
}
```

```typescript
const example = (): void => {};
```

```bash
make test
```

```

### Common Languages

- `go` - Go code
- `typescript` or `tsx` - TypeScript/React
- `javascript` or `jsx` - JavaScript/React
- `bash` - Shell commands
- `yaml` - YAML configuration
- `json` - JSON data
- `markdown` - Markdown examples

### Inline Code

Use backticks for inline code references:

```markdown
Use `fmt.Errorf` with the `%w` verb for error wrapping.
Call `useEffect` hook for side effects.
Run `make test` to execute tests.
```

## Emphasis

### Bold for Key Terms

```markdown
- **Error wrapping**: Preserves error chain
- **Token budget**: 400-600 tokens per file
```

Use bold for labels, key concepts, and important terms.

### Italic for Emphasis

```markdown
Extract *actual* patterns from code, not aspirational ones.
Documentation reflects *what is*, not what should be.
```

Use italic sparingly for emphasis within sentences.

## Lists

### Unordered Lists

Use for non-sequential items:

```markdown
- First item
- Second item
- Third item
```

### Ordered Lists

Use for sequential steps or rankings:

```markdown
1. First step
2. Second step
3. Third step
```

### Nested Lists

Indent with two spaces:

```markdown
- Top level
  - Nested item
  - Another nested item
- Another top level
```

### Definition Lists (Bold Pattern)

```markdown
- **Term**: Definition or description
- **Another term**: Another definition
```

This pattern is used extensively in "Why Use This Pattern" sections.

## Links

### Inline Links

```markdown
See [Error Wrapping](https://go.dev/blog/go1.13-errors) for details.
```

### Reference to Files

```markdown
See `.ai/common-standards/error-wrapping.md` for error handling.
Load `backend/services/user_service.go` for service patterns.
```

Use backticks around file paths. Prefer relative paths from project root.

### External Documentation

```markdown
## References

- Go error wrapping: <https://go.dev/blog/go1.13-errors>
- React hooks: <https://react.dev/reference/react/hooks>
```

Use angle brackets for URLs to avoid markdown parsing issues.

## Formatting Don'ts

### Avoid

```markdown
# Avoid excessive blank lines


# Avoid trailing whitespace after lines

# Avoid mixing list styles
- Bullet
1. Number
- Bullet

# Avoid emphasis overuse
**This** is **too** much **bold** text.
```

### Instead

```markdown
# Use single blank lines between sections

# Keep lines clean (no trailing spaces)

# Use consistent list style per section
- Bullet
- Bullet
- Bullet

# Use emphasis strategically
**Key term** stands out in regular text.
```

## Special Sections

### Pattern Section

Always use triple backticks with language:

```go
if err != nil {
    return fmt.Errorf("operation failed: %w", err)
}
```

### Why Section

Use bold definition list:

```markdown
- **Benefit 1**: Clear explanation
- **Benefit 2**: Another benefit
```

### Examples Section

Include file references as comments:

```go
// backend/services/user.go:42
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}
```

## Token Optimization

These conventions minimize tokens:

1. **Single blank lines**: Don't use multiple blank lines
2. **No trailing whitespace**: Wastes tokens
3. **Backticks for code**: More token-efficient than bold
4. **Brief headings**: State content directly
5. **Structured lists**: More scannable than paragraphs

## References

- CommonMark: <https://commonmark.org/>
- Token efficiency: `.ai/common-standards/token-efficient-writing.md`
- Pattern structure: `.ai/common-standards/pattern-documentation.md`
