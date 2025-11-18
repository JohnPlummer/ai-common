# GitHub Flow - Branch-Based Development

*Category: workflow*

Feature branch workflow preventing direct main commits, ensuring code review and CI verification.

## Pattern

```bash
# Create feature branch
git checkout -b ST-XXX-brief-description

# Commit changes
git add .
git commit -m "feat(scope): description"

# Push and create PR
git push -u origin ST-XXX-brief-description
gh pr create --title "ST-XXX: Summary" --body "..."
```

## Why Use This Pattern

- **Code review**: All changes reviewed before merge
- **CI verification**: Tests and checks run on every PR
- **Clear history**: Feature branches provide change context
- **Safe rollback**: Easy to revert merged feature branches
- **Prevents accidents**: Pre-commit hooks block direct main commits

## Branch Naming

With ticket: `ST-XXX-brief-description` or `fix/ST-XXX-brief-description`

Without ticket: `feature/description`, `fix/description`, `chore/description`

## Commit Messages

Format: `type(scope): description`

Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`

Examples:

```bash
feat(auth): add JWT token validation
fix(api): handle null response
chore(deps): update dependencies
```

## Pre-Commit Hook

Prevent accidental main commits:

```bash
# .git/hooks/pre-commit
#!/bin/bash
branch="$(git rev-parse --abbrev-ref HEAD)"
if [ "$branch" = "main" ]; then
  echo "ERROR: Direct commits to main not allowed"
  exit 1
fi
```

Make executable: `chmod +x .git/hooks/pre-commit`

## Pull Request Format

**Title**: `ST-XXX: Brief summary`

**Body**:

```markdown
## Summary
- Key changes

## Testing
- [ ] All tests pass
- [ ] Manual testing completed

## Related
- Closes: ST-XXX
```

## Recovery from Accidental Main Commit

Create retroactive feature branch:

```bash
# Create branch from parent
git checkout -b ST-XXX-description main~1

# Cherry-pick the commit
git cherry-pick <commit-hash>

# Push and create PR
git push -u origin ST-XXX-description
gh pr create
```

## Related Standards

- `workflow/jira-workflow.md` - Ticket implementation workflow with feature branches
