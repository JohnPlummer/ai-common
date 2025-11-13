# Code References in Documentation

Standard format for referencing real code from the codebase in pattern documentation.

## Pattern

```go
// backend/services/user_service.go:42
if err != nil {
    return fmt.Errorf("failed to create user: %w", err)
}
```

Format: `file/path/from/root.ext:line_number` as comment above code.

## Why Use This Pattern

- **Verifiable**: References can be checked against actual code
- **Contextual**: Developers know exact location for more context
- **Maintainable**: Easy to update when code changes
- **Trustworthy**: Shows pattern is used in production, not theoretical
- **Navigable**: File paths enable quick navigation in editors

## File Path Format

### From Project Root

```go
// backend/pkg/api/repositories/postgres_activity_repository.go:54
if err != nil {
    return nil, fmt.Errorf("failed to query activities: %w", err)
}
```

Always use paths relative to project root, not absolute system paths.

### Line Numbers

Include specific line number where pattern starts:

```typescript
// frontend/src/tests/unit/router.test.tsx:24
describe('Application Routing', () => {
  it('should render Home page at /', async () => {
    // test code
  });
});
```

Line numbers help locate exact pattern instance. Update when code moves.

### Multiple Examples

```go
// Example 1: Service layer
// backend/services/content_service.go:123
if err := s.processContent(ctx, content); err != nil {
    return fmt.Errorf("failed to process content: %w", err)
}

// Example 2: Repository layer
// backend/repositories/activity_repo.go:78
if err := r.db.Exec(ctx, query, args...); err != nil {
    return fmt.Errorf("failed to insert activity: %w", err)
}
```

Use descriptive comments to distinguish examples. Show pattern in different contexts.

## Context Inclusion

### Minimal Context

Include just enough code to show the pattern:

```go
// pipeline/pkg/workers/reddit_worker.go:196
if err != nil {
    return fmt.Errorf("failed to fetch posts from r/%s: %w", w.subreddit, err)
}
```

Typically 3-7 lines. Focus on the pattern itself.

### Extended Context

When pattern depends on surrounding code:

```go
// backend/services/user_service.go:89
func (s *UserService) CreateUser(ctx context.Context, req *CreateUserRequest) (*User, error) {
    user, err := s.repo.Create(ctx, req)
    if err != nil {
        return nil, fmt.Errorf("failed to create user: %w", err)
    }
    return user, nil
}
```

Show function signature when pattern context matters. Typically 5-15 lines.

### What to Omit

```go
// Good: Shows relevant pattern
// backend/services/activity_service.go:45
if err != nil {
    return fmt.Errorf("failed to save activity: %w", err)
}

// Avoid: Includes irrelevant details
// Don't include:
// - Import statements
// - Unrelated function code
// - Long comment blocks
// - Multiple unrelated patterns
```

Include only code relevant to the pattern being documented.

## Examples from Different Files

### Go Backend

```go
// backend/pkg/api/handlers/activity_handler.go:67
func (h *ActivityHandler) GetActivities(w http.ResponseWriter, r *http.Request) {
    activities, err := h.service.List(r.Context())
    if err != nil {
        h.logger.Error("failed to list activities", "error", err)
        http.Error(w, "Internal Server Error", http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(activities)
}
```

### TypeScript Frontend

```typescript
// frontend/src/components/ActivityCard.tsx:12
export const ActivityCard: React.FC<ActivityCardProps> = ({ activity }) => {
  return (
    <Card>
      <CardContent>
        <Typography variant="h5">{activity.title}</Typography>
        <Typography>{activity.description}</Typography>
      </CardContent>
    </Card>
  );
};
```

### Tests

```go
// backend/tests/integration/services/user_service_test.go:34
It("should create user successfully", func() {
    user, err := service.CreateUser(ctx, validRequest)
    Expect(err).NotTo(HaveOccurred())
    Expect(user.Email).To(Equal(validRequest.Email))
})
```

## Reference Stability

### Version Control Awareness

Line numbers change as code evolves. When referencing:

1. **Check current state**: Verify line number is accurate
2. **Use stable references**: Reference committed code, not work in progress
3. **Update documentation**: When code moves significantly, update references
4. **Consider alternatives**: For frequently changing code, reference by function name

### Alternative: Function References

```markdown
See `CreateUser` function in `backend/services/user_service.go`
```

When exact line numbers are too volatile, reference by function name instead.

## Linking to External Documentation

### Official Documentation

```markdown
## References

- Go error wrapping: <https://go.dev/blog/go1.13-errors>
- React hooks: <https://react.dev/reference/react/hooks>
- Ginkgo BDD: <https://onsi.github.io/ginkgo/>
```

Use angle brackets around URLs. Link to official docs for authoritative references.

### API Documentation

```markdown
See `pgx.Pool` documentation: <https://pkg.go.dev/github.com/jackc/pgx/v5/pgxpool>
```

Link to package documentation for third-party libraries.

### Internal Documentation

```markdown
See architecture overview: `docs/architecture/overview.md`
See testing strategy: `.ai/project-standards/bdd-testing.md`
```

Use relative paths for internal docs. Prefer backticks around paths.

## Guidelines

1. **Always use project-relative paths**: From repository root
2. **Include line numbers**: Helps locate code quickly
3. **Keep context minimal**: 3-7 lines typical, 15 lines maximum
4. **Verify references**: Check code actually exists at that location
5. **Use comments**: Format as language-appropriate comment
6. **Update references**: When code moves, update documentation
7. **Show real code**: Never fabricate or modify examples

## Anti-patterns

```markdown
// Bad: Absolute system path
// /Users/dev/projects/repo/backend/service.go:42

// Bad: No line number
// backend/service.go
if err != nil { ... }

// Bad: Too much context (50+ lines)
// backend/service.go:10
func VeryLongFunction() {
  // ... 50 lines of code
}

// Good: Project-relative with line number
// backend/service.go:42
if err != nil {
    return fmt.Errorf("failed: %w", err)
}
```

## Related Patterns

- Pattern structure: `.ai/common-standards/pattern-documentation.md`
- Token efficiency: `.ai/common-standards/token-efficient-writing.md`
- Markdown formatting: `.ai/common-standards/markdown-conventions.md`
