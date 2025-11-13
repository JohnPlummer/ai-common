# Task Decomposition

Task sizing and decomposition criteria for maintaining focused, manageable development work.

## Pattern

```
Optimal task size:
- Duration: 15-40 minutes
- Files: 3-4 files maximum
- Scope: Single architectural layer
- Dependencies: Minimal cross-references
```

If task exceeds limits, decompose into smaller subtasks.

## Why Use This Pattern

- **Maintainable focus**: Shorter tasks reduce cognitive load
- **Faster feedback**: Complete and verify work quickly
- **Easier review**: Smaller changes are easier to review
- **Better testing**: Focused scope means targeted tests
- **Reduced conflicts**: Smaller changes minimize merge conflicts

## Task Sizing Criteria

### Duration Limit: 15-40 Minutes

```markdown
Good: Implement user login endpoint (30 min)
- Handler function
- Service method
- Repository query
- Tests

Too large: Implement entire authentication system (4 hours)
Split into:
- Login endpoint (30 min)
- Registration endpoint (30 min)
- Password reset endpoint (30 min)
- JWT middleware (20 min)
```

### File Limit: Maximum 3-4 Files

```markdown
Good: Add location filter (3 files)
- repository.go (add query method)
- service.go (add filter logic)
- repository_test.go (add tests)

Too large: Refactor entire repository layer (15 files)
Split by: One repository at a time
```

### Focus: Single Architectural Layer

```markdown
Good: Implement repository method
- Just data access layer
- Tests for that layer

Too broad: Feature spanning UI → API → Database
Split into:
- Backend API endpoint
- Database repository method
- Frontend component
```

### Context: Minimize Dependencies

```markdown
Good: Add validation to existing endpoint
- Focused change
- Clear boundaries
- Few dependencies

Too coupled: Change requiring updates across 5 different services
Split into: Incremental changes with backward compatibility
```

## Before Starting Work

```markdown
1. Understand specific task requirements
2. Check existing patterns in related files
3. Identify which .ai/ standards to load
4. Estimate task size
5. If >40 min or >4 files: decompose into subtasks
```

## Decomposition Strategies

### By Layer

```markdown
Large feature: "Add activity recommendations"

Decompose:
1. Database schema and repository (30 min)
2. Service layer logic (35 min)
3. API endpoint (25 min)
4. Frontend component (30 min)
```

### By Component

```markdown
Large refactor: "Update error handling across backend"

Decompose:
1. Location service errors (30 min)
2. Activity service errors (30 min)
3. User service errors (25 min)
4. HTTP middleware errors (20 min)
```

### By Functionality

```markdown
Large feature: "User dashboard"

Decompose:
1. Fetch user activities (25 min)
2. Display activities list (30 min)
3. Add filter controls (35 min)
4. Add pagination (20 min)
```

## Red Flags for Decomposition

Task needs decomposition if:

- Estimated duration >40 minutes
- Touching >4 files
- Spanning multiple architectural layers
- Multiple unrelated changes bundled together
- Can't write clear, focused commit message

## Guidelines

1. **Start small**: Underestimate rather than overestimate
2. **One layer at a time**: Don't span architecture boundaries
3. **Complete each task**: Finish, test, commit before next task
4. **Incremental value**: Each subtask should be potentially shippable
5. **Clear boundaries**: Each subtask should have obvious start/end

## Related Patterns

- BDD testing: `.ai/project-standards/bdd-testing.md`
- Clean architecture: `.ai/project-standards/clean-architecture.md`
