---
name: dev
description: >
  Development workflow skills for requirement-driven development.
  Routes to: planner, doer, save-wip, shipper, fail, track, gh-remote.
  Triggers on: "plan this", "do REQ", "save progress", "ship REQ", "fail REQ",
  "track changes", "gh remote", or any REQ-NNNN reference.
---

# Skill: dev

Router for waramity development workflow operations.

## Sub-Skills

| Sub-Skill | Purpose | Trigger Phrases |
|-----------|---------|-----------------|
| **planner** | Analyze task, create REQ file | "plan this", "analyze this", "let's plan" |
| **doer** | Implement a planned REQ | "do REQ", "implement REQ-NNNN", "work on REQ" |
| **save-wip** | Commit unfinished work | "save progress", "checkpoint", "push WIP" |
| **shipper** | Commit & push completed REQ | "ship REQ", "commit and push", "finish REQ" |
| **fail** | Revert and abandon REQ | "fail REQ", "revert REQ", "abandon REQ" |
| **track** | Show uncommitted files | "track changes", "what should I commit" |
| **gh-remote** | GitHub ops via gh CLI only | "gh remote", "do on remote", "via gh" |

---

## Routing Logic

### Route to `planner` when:
- User provides code, task, or feature idea to plan
- User says "plan this", "analyze this", "what do I need"
- User pastes code and wants structured planning before coding
- Creating a new REQ file in `.waramity/dev/requirement/`

### Route to `doer` when:
- User references a specific REQ number (e.g., "REQ-0001")
- User says "do REQ", "implement", "work on"
- REQ file exists with status `🟡 Planned` or `🚧 WIP`

### Route to `save-wip` when:
- User wants to save incomplete/broken code
- User says "checkpoint", "save progress", "push WIP"
- Code may have bugs but needs to be preserved

### Route to `shipper` when:
- User wants to commit and push completed work
- User says "ship REQ", "commit and push", "finish"
- REQ has status `⏳ Wait to Review`

### Route to `fail` when:
- User wants to abandon/revert a REQ
- User says "fail REQ", "revert", "abandon", "scrap"
- Reverting code changes and deleting REQ/WIP files

### Route to `track` when:
- User wants to see uncommitted changes
- User says "track changes", "what should I commit", "pending changes"
- Analyzing working tree for commit suggestions

### Route to `gh-remote` when:
- User wants GitHub operations without local git
- User says "gh remote", "do on remote", "via gh", "no local"
- Operating on PRs, issues, branches remotely

---

## Workflow Lifecycle

```
┌─────────┐     ┌───────┐     ┌──────────┐     ┌─────────┐
│ planner │ ──▶ │ doer  │ ──▶ │ shipper  │ ──▶ │  Done   │
└─────────┘     └───────┘     └──────────┘     └─────────┘
     │               │              │
     │               ▼              │
     │          ┌──────────┐        │
     │          │ save-wip │        │
     │          └──────────┘        │
     │               │              │
     ▼               ▼              ▼
┌─────────────────────────────────────────┐
│                 fail                    │
└─────────────────────────────────────────┘
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Plan a new task | `/waramity` → "plan this: ..." |
| Implement REQ | `/waramity` → "do REQ-0001" |
| Save unfinished work | `/waramity` → "save my progress" |
| Ship completed REQ | `/waramity` → "ship REQ-0001" |
| Abandon/revert REQ | `/waramity` → "fail REQ-0001" |
| Check pending changes | `/waramity` → "track changes" |
| GitHub remote ops | `/waramity` → "gh remote: view PR #123" |

---

## Rules

- Always route to the appropriate sub-skill based on user intent
- If user mentions REQ-NNNN, check if it exists before routing
- If unclear between planner and doer, ask: "Do you want to (1) plan this task, or (2) implement an existing REQ?"
- Never perform destructive operations (fail, force push) without confirmation
