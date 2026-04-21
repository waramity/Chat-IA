---
name: init
description: Initialize the .waramity/ folder structure (dev, business, design) in the current project, and optionally pull the latest waramity skill version from the remote repository. Trigger on phrases like "init waramity", "setup waramity", "initialize .waramity", "pull latest skill", "update waramity skill", "setup project", or when the user wants to start using waramity in a new project.
---

# Skill: init

Bootstrap the `.waramity/` folder structure for the current project and optionally sync the latest waramity skills from the remote repository.

## Workflow

### Step 1: Check Existing State

```bash
ls -la .waramity/ 2>/dev/null && echo "EXISTS" || echo "NOT_FOUND"
```

- If `.waramity/` already exists, show the current structure and ask: "`.waramity/` already exists. Do you want to (1) add missing folders only, (2) pull latest skill version, or (3) both?"
- If not found, proceed to Step 2.

---

### Step 2: Create Folder Structure

Create all required directories:

```bash
mkdir -p .waramity/dev/requirement
mkdir -p .waramity/dev/wip
mkdir -p .waramity/business
mkdir -p .waramity/design
echo ".waramity folder initialized."
ls -la .waramity/
```

Expected structure:
```
.waramity/
├── business/          # Business documents and requirements
├── design/            # Design assets and specifications
└── dev/
    ├── requirement/   # Planned REQ files (REQ-NNNN-slug.md)
    └── wip/           # Work-in-progress files (WIP-NNNN-slug.md)
```

Confirm success: "`.waramity/` structure created with `business/`, `design/`, and `dev/` folders."

---

### Step 3: Download Karpathy-Style CLAUDE.md Guidelines

Download the Andrej Karpathy coding guidelines to the project's `.claude/CLAUDE.md`:

```bash
mkdir -p .claude
gh api repos/forrestchang/andrej-karpathy-skills/contents/CLAUDE.md -q .content \
  | base64 -d > .claude/CLAUDE.md \
  && echo "Downloaded: .claude/CLAUDE.md (Karpathy guidelines)" \
  || echo "Failed to download CLAUDE.md"
```

This provides 4 core principles:
1. **Think Before Coding** - Surface assumptions and tradeoffs
2. **Simplicity First** - Minimum code, nothing speculative
3. **Surgical Changes** - Only necessary edits, no cosmetic improvements
4. **Goal-Driven Execution** - Define success criteria, loop until verified

---

### Step 4: Pull Latest Skill Version (Optional)

Ask the user:
> "Do you also want to pull the latest waramity skill from `waramity/waramity-skills`? This will update your local `~/.claude/skills/waramity/` files."

If yes, download all skill files from GitHub using `gh` CLI:

```bash
SKILLS_DIR="$HOME/.claude/skills/waramity"
REPO="waramity/waramity-skills"
REMOTE_BASE="skills/waramity"

# Download root SKILL.md
gh api "repos/$REPO/contents/$REMOTE_BASE/SKILL.md" -q .content \
  | base64 -d > "$SKILLS_DIR/SKILL.md"

# Download init skill (at root level)
mkdir -p "$SKILLS_DIR/init"
gh api "repos/$REPO/contents/$REMOTE_BASE/init/SKILL.md" -q .content 2>/dev/null \
  | base64 -d > "$SKILLS_DIR/init/SKILL.md" \
  && echo "Updated: init/SKILL.md" \
  || echo "Skipped (not found): init/SKILL.md"

# Download each dev sub-skill SKILL.md
for sub in doer fail gh-remote planner save-wip shipper track; do
  mkdir -p "$SKILLS_DIR/dev/$sub"
  gh api "repos/$REPO/contents/$REMOTE_BASE/dev/$sub/SKILL.md" -q .content 2>/dev/null \
    | base64 -d > "$SKILLS_DIR/dev/$sub/SKILL.md" \
    && echo "Updated: dev/$sub/SKILL.md" \
    || echo "Skipped (not found): dev/$sub/SKILL.md"
done
```

Confirm: "Waramity skill updated to latest version from `waramity/waramity-skills`."

---

### Step 5: Present Summary

After completing, show:

```
Initialization complete:

  Project .waramity/:
    OK .waramity/business/         -- store business documents here
    OK .waramity/design/           -- store design assets here
    OK .waramity/dev/requirement/  -- store planned REQ files here
    OK .waramity/dev/wip/          -- store WIP checkpoints here

  Guidelines:
    OK .claude/CLAUDE.md           -- Karpathy-style coding principles

  Next steps:
    - Plan a task:  /waramity  -> "plan this: <description>"
    - Do a REQ:     /waramity  -> "do REQ-NNNN"
    - Save WIP:     /waramity  -> "save my progress"
    - Ship a REQ:   /waramity  -> "ship REQ-NNNN"
```

---

## Rules

- Never overwrite existing files in `.waramity/` -- only create missing directories.
- Always use `rtk gh` prefix for remote operations per CLAUDE.md golden rule.
- Never pull skill files into the project directory -- skills always go to `~/.claude/skills/waramity/`.
- If `gh` CLI is unavailable, skip the pull step and inform the user.

---

## Examples

### Initialize a fresh project
```bash
mkdir -p .waramity/dev/requirement .waramity/dev/wip .waramity/business .waramity/design
ls -la .waramity/
```

### Pull latest skill only (no folder init needed)
```bash
gh api repos/waramity/waramity-skills/contents/skills/waramity/SKILL.md -q .content \
  | base64 -d > ~/.claude/skills/waramity/SKILL.md
```
