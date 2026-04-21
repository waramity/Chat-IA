---
name: setup
description: >
  Setup and maintenance utilities for waramity skills.
  Routes to: init (project setup), sync (sync with remote), update (validate references).
  Triggers on: "setup waramity", "init project", "sync skills", "pull skills", "push skills",
  "check skills", "validate skills", "fix skill paths", "initialize waramity".
---

# Skill: setup

Router for waramity setup and maintenance operations.

## Sub-Skills

| Sub-Skill | Purpose | Trigger Phrases |
|-----------|---------|-----------------|
| **init** | Initialize `.waramity/dev/` folder in project | "init waramity", "setup project", "initialize" |
| **sync** | Sync skills with GitHub remote | "sync skills", "pull skills", "push skills", "upload to github", "download skills" |
| **update** | Validate and fix skill references | "check skills", "validate skills", "fix skill paths", "skill integrity", "update references" |

---

## Routing Logic

### Route to `init` when:
- User wants to set up a new project with waramity
- User mentions "init", "initialize", "setup project"
- `.waramity/dev/` folder doesn't exist and user wants to create it
- User wants to pull latest skill version during initial setup

### Route to `sync` when:
- User wants to sync skills between local and remote
- User mentions "pull", "push", "sync skills"
- User wants to upload local changes to GitHub
- User wants to download latest skills from GitHub
- User asks for "status" to compare local vs remote

### Route to `update` when:
- User wants to check skill integrity
- User mentions "check skills", "validate", "fix paths"
- User wants to find broken references in routers
- User asks "what needs updating" in skill files

---

## Quick Reference

### Initialize a new project
```
/waramity → "init waramity" or "setup project"
```
Creates:
```
.waramity/
└── dev/
    ├── requirement/
    └── wip/
```

### Sync with remote
```
/waramity → "sync skills pull"   # Download from GitHub
/waramity → "sync skills push"   # Upload to GitHub
/waramity → "sync status"        # Compare without changes
```

### Validate skill references
```
/waramity → "check skills"       # Show issues only
/waramity → "fix skill paths"    # Show and offer to fix
```

---

## Rules

- Always route to the appropriate sub-skill based on user intent
- If unclear, ask: "Do you want to (1) initialize a project, (2) sync skills with GitHub, or (3) check/fix skill references?"
- Never perform multiple operations without explicit user request
