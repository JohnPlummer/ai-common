# Pattern Documentation Structure

Standard structure for documenting code patterns in AI-optimized documentation.

## Pattern

```markdown
# Pattern Name

Brief description in one sentence.

## Pattern

```language
// Minimal code example showing the pattern
```

## Why Use This Pattern

- **Benefit 1**: Explanation
- **Benefit 2**: Explanation
- **Benefit 3**: Explanation

## Examples from Codebase

### Context Description

```language
// file/path.ext:line_number
actual code from codebase
```

## Guidelines

1. **Guideline 1**: When to use this approach
2. **Guideline 2**: Best practices
3. **Guideline 3**: Common considerations

## Anti-patterns

```language
// Bad: Explanation of what's wrong
bad example

// Good: Explanation of correct approach
correct example
```

## Related Patterns

- Reference to related documentation files

```

## Why Use This Structure

- **Scannable**: AI can quickly locate relevant sections
- **Example-driven**: Code examples are primary teaching tool
- **Contextual**: Real codebase references provide concrete context
- **Token-efficient**: Structured format minimizes redundant text
- **Actionable**: Guidelines and anti-patterns provide clear direction

## Section Breakdown

### Title and Description
One-sentence description states what the pattern is. Avoids verbose introductions.

### Pattern Section
Minimal code example (3-10 lines) showing the pattern essence. Must be valid, compilable code.

### Why Use This Pattern
3-5 bullet points with bold labels. Each bullet explains one benefit. Keeps benefits concrete and measurable.

### Examples from Codebase
2-3 real examples with file paths and line numbers. Shows pattern in actual use. Format: `file/path.ext:line_number` as comment above code.

### Guidelines
Numbered list of best practices. Focus on when and how to apply the pattern. Typically 3-5 items.

### Anti-patterns
Side-by-side comparison of bad and good approaches. Uses comments to explain the issue. Shows what to avoid.

### Related Patterns
Links to other documentation files. Keeps related concepts connected.

## Examples Following This Structure

All current common-standards files follow this structure:

- `error-wrapping.md` - Error handling pattern
- `testing-patterns.md` - Vitest testing approach
- `constructors.md` - Constructor pattern
- `react-hooks.md` - React hooks usage

## Token Budget

Target 400-600 tokens per pattern file:

- Title and description: 20-30 tokens
- Pattern example: 50-100 tokens
- Why section: 80-120 tokens
- Real examples: 150-200 tokens
- Guidelines: 60-80 tokens
- Anti-patterns: 80-100 tokens
- Related: 20-30 tokens

## Writing Guidelines

1. **Start with code**: Lead with examples, not theory
2. **Use actual code**: Extract from real files, include file:line references
3. **Be specific**: Concrete benefits over abstract principles
4. **Stay focused**: One pattern per file
5. **Minimize prose**: Let code speak, add only essential explanation
6. **Use bullets**: Easier to scan than paragraphs
7. **Format consistently**: Follow structure exactly

## References

- Token efficiency: `.ai/common-standards/token-efficient-writing.md`
- Code references: `.ai/common-standards/code-references.md`
