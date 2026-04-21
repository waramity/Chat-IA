---
name: track
description: Track uncommitted files and show a structured commit list. Analyzes working tree changes and suggests logical commit groupings. Trigger on phrases like "track changes", "what should I commit", "show uncommitted", "commit list", "pending changes".
---

# Skill: track

Analyze uncommitted files and suggest a structured commit plan.

## Trigger Phrases

- "track changes"
- "what should I commit"
- "show uncommitted"
- "commit list"
- "pending changes"
- "what's changed"

## Workflow

### Step 1: Gather Status

```bash
rtk git status --porcelain
```

Parse output codes:
- `??` — Untracked (new file)
- ` M` — Modified (unstaged)
- `M ` — Modified (staged)
- `A ` — Added (staged)
- `D ` — Deleted (staged)
- ` D` — Deleted (unstaged)
- `R ` — Renamed
- `AM` — Added then modified

### Step 2: Group by Category

Organize files into logical commit groups:

| Category | Pattern | Suggested Commit Type |
|----------|---------|----------------------|
| Features | `src/`, `lib/`, `app/` | `feat:` |
| Tests | `test/`, `spec/`, `*.test.*` | `test:` |
| Docs | `*.md`, `docs/` | `docs:` |
| Config | `.*rc`, `*.config.*`, `*.json` | `chore:` |
| Skills | `.claude/skills/` | `chore(skills):` |
| Styles | `*.css`, `*.scss`, `*.styled.*` | `style:` |
| Build | `Makefile`, `Dockerfile`, `*.yml` | `build:` |

### Step 3: Output Commit Plan

Display a structured commit plan:

```
## Uncommitted Changes Summary

### 📁 Untracked Files (N files)
- path/to/file1
- path/to/file2

### ✏️ Modified Files (N files)  
- path/to/file3
- path/to/file4

### 🗑️ Deleted Files (N files)
- path/to/file5

---

## Suggested Commits

### Commit 1: `feat: add new feature X`
Files:
- src/feature.ts
- src/utils/helper.ts

### Commit 2: `test: add tests for feature X`
Files:
- test/feature.test.ts

### Commit 3: `docs: update README`
Files:
- README.md

---

**Total:** N commits covering M files
```

### Step 4: Offer Actions

After showing the plan, offer:

1. **Proceed with commits** → Hand off to shipper or save-wip
2. **Modify grouping** → Let user adjust commit boundaries
3. **Show diff** → Run `rtk git diff` on specific files
4. **Cancel** → Exit without action

## Rules

1. **Always use `rtk` prefix** per CLAUDE.md
2. **Never auto-commit** — only suggest, let user confirm
3. **Group logically** — related changes in same commit
4. **Use conventional commits** — `type(scope): message`
5. **Show full paths** — no ambiguity

## Examples

### Basic usage
```
User: track changes
Assistant: [runs git status, groups files, shows commit plan]
```

### After coding session
```
User: what should I commit
Assistant: [analyzes changes, suggests 3 commits: feat, test, docs]
```
