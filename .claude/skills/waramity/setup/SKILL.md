---
name: setup
description: >
  Setup and maintenance utilities for waramity skills.
  Routes to: init (project setup), sync (sync with remote), update (validate references),
  connect (link waramity skills to external skills), install-skill (install from registry).
  Triggers on: "setup waramity", "init project", "sync skills", "pull skills", "push skills",
  "check skills", "validate skills", "fix skill paths", "initialize waramity",
  "connect skill", "link skill", "integrate skill", "install skill", "list skills", "available skills".
---

# Skill: setup

Router for waramity setup and maintenance operations.

## Sub-Skills

| Sub-Skill | Purpose | Trigger Phrases |
|-----------|---------|-----------------|
| **init** | Initialize `.waramity/dev/` folder in project | "init waramity", "setup project", "initialize" |
| **sync** | Sync skills with GitHub remote | "sync skills", "pull skills", "push skills", "upload to github", "download skills" |
| **update** | Validate and fix skill references | "check skills", "validate skills", "fix skill paths", "skill integrity", "update references" |
| **connect** | Link waramity skills to external skills | "connect skill", "link skill", "integrate skill", "connect to", "link waramity to" |
| **install-skill** | Install skills from registry | "install skill", "list skills", "available skills", "add skill", "uninstall skill" |

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

### Route to `connect` when:
- User wants to link a waramity skill to an external skill
- User mentions "connect", "link", "integrate" with skill references
- User wants to create cross-skill documentation
- User mentions external skill paths (e.g., "ui-ux-pro-max")

### Route to `install-skill` when:
- User wants to see available skills to install
- User mentions "install skill", "list skills", "available skills"
- User wants to add a new skill from the registry
- User wants to uninstall a skill
- User asks "what skills can I install"

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

### Connect to external skills
```
/waramity → "connect design to ui-ux-pro-max"
```
Creates:
```
.waramity/
└── memory/
    └── connected-skill/
        └── design--ui-ux-pro-max.md
```

### Install skills from registry
```
/waramity → "list skills"           # Show available skills
/waramity → "install ui-ux-pro-max" # Install and auto-connect
/waramity → "show installed"        # List installed skills
/waramity → "uninstall skill-name"  # Remove a skill
```
Creates:
```
.waramity/
└── memory/
    └── installed-skill/
        └── ui-ux-pro-max.md
.claude/
└── skills/
    └── external/
        └── ui-ux-pro-max/
```

---

## Rules

- Always route to the appropriate sub-skill based on user intent
- If unclear, ask: "Do you want to (1) initialize a project, (2) sync skills with GitHub, (3) check/fix skill references, (4) connect to an external skill, or (5) install skills from registry?"
- Never perform multiple operations without explicit user request
