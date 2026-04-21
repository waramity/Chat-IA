---
name: init
description: Initialize the .waramity folder structure in the current project, and optionally pull the latest waramity skill version from the remote repository. Trigger on phrases like "init waramity", "setup waramity", "initialize .waramity", "pull latest skill", "update waramity skill", "setup project", or when the user wants to start using waramity in a new project.
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
mkdir -p .waramity/requirement
mkdir -p .waramity/wip
echo ".waramity folder initialized."
ls -la .waramity/
```

Expected structure:
```
.waramity/
├── requirement/   # Planned REQ files (REQ-NNNN-slug.md)
└── wip/           # Work-in-progress files (WIP-NNNN-slug.md)
```

Confirm success: "`.waramity/` structure created with `requirement/` and `wip/` folders."

---

### Step 3: Pull Latest Skill Version (Optional)

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

# List and download each sub-skill SKILL.md
for sub in doer fail gh-remote init planner save-wip shipper; do
  mkdir -p "$SKILLS_DIR/$sub"
  gh api "repos/$REPO/contents/$REMOTE_BASE/$sub/SKILL.md" -q .content 2>/dev/null \
    | base64 -d > "$SKILLS_DIR/$sub/SKILL.md" \
    && echo "Updated: $sub/SKILL.md" \
    || echo "Skipped (not found): $sub/SKILL.md"
done
```

Confirm: "Waramity skill updated to latest version from `waramity/waramity-skills`."

---

### Step 4: Present Summary

After completing, show:

```
Initialization complete:

  Project .waramity/:
    OK .waramity/requirement/  -- store planned REQ files here
    OK .waramity/wip/          -- store WIP checkpoints here

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
mkdir -p .waramity/requirement .waramity/wip
ls -la .waramity/
```

### Pull latest skill only (no folder init needed)
```bash
gh api repos/waramity/waramity-skills/contents/skills/waramity/SKILL.md -q .content \
  | base64 -d > ~/.claude/skills/waramity/SKILL.md
```
