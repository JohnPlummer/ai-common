# AI Common Standards

Shared standards, patterns, and prompts for AI assistants across all projects.

## Purpose

This repository contains portable, project-agnostic development standards that can be symlinked into any project's `.ai/` folder. The goal is to maintain consistent patterns across all projects while allowing each project to have its own specific standards.

## Theory: Progressive Loading for AI Assistants

### The Problem

AI assistants working with codebases face several challenges:

1. **Token limits**: Cannot load entire codebases at once
2. **Context relevance**: Need the right information at the right time
3. **Duplication**: Same patterns repeated across projects waste tokens
4. **Staleness**: Documentation becomes outdated as code evolves

### The Solution: Progressive Loading Map

The `.ai/` folder provides a **navigation system** for AI assistants, similar to a table of contents. Instead of loading everything upfront:

1. **Start with the map**: `llms.md` provides an overview
2. **Load on-demand**: Only load files relevant to current task
3. **Common + Project**: Combine shared patterns with project-specific details
4. **Stay current**: Documentation lives close to code, easier to keep updated

### Three-Tier Architecture

**Tier 1: Entry Points** (Always load)

- `CLAUDE.md` or `.cursorrules` - Project configuration for AI assistant
- `.ai/llms.md` - Project-specific loading map

**Tier 2: Common Standards** (Load by category)

- `~/code/ai-common/` - Shared across ALL projects via symlink
- Organized by domain: go/, typescript/, testing/, architecture/, etc.
- Updated once, benefits all projects

**Tier 3: Project Standards** (Load by need)

- `.ai/project-standards/` - Specific to THIS project only
- Project config, project errors, project workflows
- Not shared across projects

### Working Memory Files

**context.md** (Volatile - Gitignored)

- Current sprint/tickets being worked on
- Recent discoveries (last 2-4 weeks)
- Active investigations
- Cleared when no longer relevant

**memory.md** (Stable - Gitignored)

- Persistent gotchas and lessons learned
- Architecture decisions and rationale
- Troubleshooting patterns that recur
- Never cleared, only grows

**tasks/** (Scratchpad - Gitignored)

- Planning documents for active work
- Analysis notes and research
- Task breakdowns
- Deleted when task completes

### Token Efficiency

Each standard file: 400-600 tokens (300-800 words)

Why this matters:

- AI can load 5-10 standards per task (3000-5000 tokens)
- Remaining context budget for actual code
- No wasted tokens on irrelevant patterns
- Can reload different standards as task evolves

### Benefits

1. **Consistency**: Same patterns enforced across all projects
2. **Maintainability**: Update standards once, applies everywhere
3. **Reusability**: Don't duplicate common patterns in each project
4. **Token Efficiency**: AI assistants load common standards without duplication
5. **Quality**: Shared standards improve over time from multiple project learnings
6. **Discoverability**: AI knows what standards exist without loading them all

## Structure

```
ai-common/
├── README.md               # This file
├── common-llms.md          # Progressive loading map for AI assistants
├── standards/
│   ├── go/                 # Go language patterns
│   ├── typescript/         # TypeScript and JavaScript patterns
│   ├── testing/            # Testing strategies and patterns
│   ├── architecture/       # Architecture patterns
│   ├── database/           # Database patterns
│   ├── infrastructure/     # Infrastructure patterns
│   └── documentation/      # Documentation patterns
├── examples/               # Shared code examples
└── prompts/                # Reusable prompts for AI assistants
```

## Setting Up .ai Folder in a New Project

### Complete Setup from Scratch

**Step 1: Create Directory Structure**

```bash
cd /path/to/your-project
mkdir -p .ai/project-standards .ai/tasks
```

**Step 2: Create Entry Point (CLAUDE.md or .cursorrules)**

Create `CLAUDE.md` at project root:

```markdown
# CLAUDE.md

Configuration for Claude Code when working with this repository.

## Load These First

**CRITICAL:** Always load these files at the start of every session:

- `agents.md` - Universal AI assistant behaviors (if you have one)
- `.ai/llms.md` - Development standards and patterns (progressive loading map)

**Load as needed:**

- `.ai/memory.md` - Stable project knowledge, architecture, troubleshooting
- `.ai/context.md` - Current active work, recent discoveries

## Project-Specific Instructions

[Add any project-specific instructions for AI assistants here]

---

For all development standards, patterns, commands, and workflows, see `.ai/llms.md` and load relevant files on-demand.
```

**Step 3: Create Symlink to Common Standards**

```bash
cd .ai
ln -s ~/code/ai-common common
```

Verify it works:

```bash
ls -la common
# Should show: common -> /Users/[you]/code/ai-common

ls common/standards/go/
# Should list: constructors.md, error-wrapping.md, jp-go-errors.md, etc.
```

**Step 3a: Link Claude Commands to Common Prompts (Optional)**

If using Claude Code with slash commands:

```bash
# From project root
mkdir -p .claude
cd .claude
ln -s ../.ai/common/prompts commands
```

This gives you access to shared prompts like:

- `/convert-to-common-standard` - Analyze if a standard should be common
- `/document-new-standard` - Document a new pattern
- `/extract-patterns` - Extract patterns from code
- `/generate-docs` - Generate documentation

Verify:

```bash
ls -la .claude/commands
# Should show: commands -> ../.ai/common/prompts
```

**Step 4: Create Project Loading Map (.ai/llms.md)**

Create `.ai/llms.md`:

```markdown
# [Project Name] - AI Documentation

Progressive loading map for AI assistants working with [Project Name].

**Entry Point**: This file should be referenced from project config files (CLAUDE.md, .cursorrules, etc.)

## Architecture Overview

- **Backend**: [Tech stack]
- **Frontend**: [Tech stack]
- **Database**: [Database info]
- **Testing**: [Testing frameworks]

## Always Load

- `.ai/llms.md` (this file)
- [Any other always-load files]

## Load for Complex Tasks

- `.ai/memory.md` - Stable project knowledge, architecture, troubleshooting
- `.ai/context.md` - Current active work (if exists and is current)

## Common Standards (Portable Patterns)

**See** `.ai/common/common-llms.md` for the complete list of common standards.

Common standards include:
- Go patterns (constructors, error-wrapping, functional-options, jp-go-* packages)
- TypeScript/React patterns (components, hooks, interfaces, testing)
- Testing patterns (BDD, testcontainers, mocking)
- Architecture patterns (clean architecture, repository pattern)
- Database patterns (transactions, jp-go-pgx-utils)
- Infrastructure patterns (circuit-breaker, retry-logic, HTTP middleware)
- Documentation patterns (markdown, READMEs, knowledge capture)

Load common standards on-demand via `.ai/common/standards/[category]/[file].md`

## Project Standards (Project-Specific)

Load based on task context:

### [Category 1]
- `.ai/project-standards/file1.md` - Description
- `.ai/project-standards/file2.md` - Description

### [Category 2]
- `.ai/project-standards/file3.md` - Description

## Loading Strategy

**Load on-demand based on task:**

See `.ai/common/common-llms.md` for common standard loading strategies.

**Project-specific loading:**

| Task Type | Load These Standards |
|-----------|---------------------|
| Task 1 | project-file1.md, common/standards/go/jp-go-errors.md |
| Task 2 | project-file2.md, common/standards/testing/bdd-testing.md |

## File Organization

[Document your .ai folder structure]
```

**Step 5: Create Working Memory Files**

Create `.ai/context.md`:

```markdown
# Project Context

*Current active work and recent discoveries. Clear when no longer relevant.*

## Active Work

*What tickets are you working on now*

- No active work currently

## Recent Discoveries

*Things learned in last 2-4 weeks that might matter soon*

- [Date]: [Discovery]

## Evaluating/Researching

*Active investigations not tied to specific ticket yet*

- None currently

## Temporary Workarounds

*Hacks in place that need proper fixes*

- None currently

---
Last updated: [Date]
```

Create `.ai/memory.md`:

```markdown
# Project Memory

*Stable project knowledge. Things that persist across sprints.*

## Architecture Decisions

### [Decision Name]
- **Date**: [Date]
- **Context**: [Why we made this decision]
- **Decision**: [What we decided]
- **Consequences**: [Impact of the decision]

## Gotchas and Lessons Learned

### [Category]
- **Issue**: [What went wrong]
- **Root Cause**: [Why it happened]
- **Solution**: [How to avoid/fix]
- **Where**: [File paths or components affected]

## Troubleshooting Patterns

### [Problem Type]
- **Symptoms**: [How you know this is happening]
- **Diagnosis**: [How to confirm the issue]
- **Fix**: [How to resolve it]

---
Last updated: [Date]
```

**Step 6: Create README (.ai/README.md)**

Create `.ai/README.md` documenting your .ai folder structure. See the example in this project's `.ai/README.md`.

**Step 7: Update .gitignore**

Add to your project's `.gitignore`:

```gitignore
# AI Working Files
.ai/context.md
.ai/memory.md
.ai/tasks/
```

**The symlink `.ai/common` should be committed** so other developers know the structure.

**Step 8: Create Project-Specific Standards (As Needed)**

Only create standards that are truly project-specific:

```bash
# Example: Project configuration
.ai/project-standards/configuration.md

# Example: Project-specific errors
.ai/project-standards/shared-errors.md

# Example: Project workflow
.ai/project-standards/jira-workflow.md
```

### Final Structure

```
your-project/
├── CLAUDE.md                   # Entry point for AI assistants
├── .gitignore                  # Ignores context.md, memory.md, tasks/
├── .claude/                    # Claude Code configuration (optional)
│   └── commands -> ../.ai/common/prompts  # Symlink to shared prompts
└── .ai/
    ├── llms.md                 # Project loading map
    ├── README.md               # Documentation about .ai folder
    ├── context.md              # Current work (gitignored)
    ├── memory.md               # Stable knowledge (gitignored)
    ├── tasks/                  # Scratchpad (gitignored)
    ├── project-standards/      # Project-specific only
    │   ├── configuration.md
    │   ├── shared-errors.md
    │   └── ...
    └── common -> ~/code/ai-common  # Symlink to shared standards
```

### Testing the Setup

Verify AI assistant can access standards:

1. Ask AI to load `.ai/llms.md`
2. Ask AI to load a common standard: `.ai/common/standards/go/jp-go-errors.md`
3. Ask AI to load a project standard: `.ai/project-standards/configuration.md`
4. Check that all paths resolve correctly

## What Goes in Common vs Project

### Common Standards (in ai-common)

Patterns that apply to ANY project:

- ✅ Language/framework patterns (Go, TypeScript, React)
- ✅ Testing strategies (BDD, mocking, integration tests)
- ✅ Architecture patterns (clean architecture, repository)
- ✅ Documentation conventions
- ✅ Code organization principles
- ✅ Generic error handling patterns
- ✅ Configuration patterns (general approach)

### Project Standards (in project/.ai/project-standards/)

Patterns specific to ONE project:

- ❌ Specific file paths (e.g., `pipeline/pkg/errors/errors.go`)
- ❌ Project-specific types (e.g., `ErrInvalidPackageType`)
- ❌ Project structure and services
- ❌ Project CloudIds, API keys, configuration
- ❌ Project-specific workflows (Jira, make targets)
- ❌ Project-specific routing or component structure

## Contributing New Standards

### 1. Determine Category

Choose the appropriate category:

- `go/` - Go language patterns
- `typescript/` - TypeScript/JavaScript/React patterns
- `testing/` - Testing strategies (any language)
- `architecture/` - Architecture patterns (any language)
- `database/` - Database patterns (any language)
- `infrastructure/` - Circuit breakers, retry, middleware
- `documentation/` - Documentation patterns

### 2. Use the Conversion Prompt

If you have a project-specific standard to convert:

```markdown
I want to analyze [filename.md] to see if it should be a common standard.
```

The AI assistant will analyze the file using `prompts/convert-to-common-standard.md` and help you split common vs project-specific parts.

### 3. Follow the Template

```markdown
# [Pattern Name]

*Category: [category name]*

## When to Use

[1-2 sentences describing when this pattern applies]

## Pattern

[Core pattern description - no project-specific details]

## Implementation

[Generic implementation steps or code examples]

## Examples

[Generic examples or reference to examples/]

## Related Standards

- [link to related standard]

## Common Pitfalls

[Issues that apply across all projects]
```

### 4. Keep It Token-Efficient

- Target 400-600 tokens (300-800 words)
- Focus on the pattern, not the explanation
- Use examples instead of lengthy descriptions
- Reference other standards rather than duplicating

### 5. Add to common-llms.md

Update the progressive loading map so AI assistants can find your standard.

## Updating Existing Standards

1. Edit the standard file in `~/code/ai-common/standards/[category]/`
2. Test in one project first
3. Push changes - all projects get the update via symlink

## Package-Specific Standards

For extracted shared packages (like jp-go-*), document them in:

- `standards/go/jp-go-errors.md` - Error handling patterns
- `standards/go/jp-go-config.md` - Configuration patterns
- `standards/go/jp-go-pgx-utils.md` - Database utilities
- etc.

These standards should show how to USE the packages, not describe their internals.

## Questions or Issues

- Check `prompts/` for helpful prompts
- Check `examples/` for code examples
- Ask your AI assistant to search common standards
- Update `common-llms.md` if structure changes

## Version History

- 2025-11-13: Initial creation with standards extracted from some-things-to-do project
- Standards include patterns from PR #91 (jp-go-* package extraction)
