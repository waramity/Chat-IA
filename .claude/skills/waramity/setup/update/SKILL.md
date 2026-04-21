---
name: update
description: >
  Validate skill routers and sync to waramity-skills GitHub README.md.
  Checks broken paths, outdated names, mismatched connections, then updates README.
  Triggers on: "check skills", "validate skills", "fix skill paths",
  "update skills readme", "sync skills to github", "push skills list".
---

# Skill: update

Validate skill router references and sync skills list to GitHub README.md.

## Purpose

1. **Validate**: Scan all skill files for broken paths, name mismatches, and broken connections
2. **Sync**: Update README.md in https://github.com/waramity/waramity-skills with current skills table

---

## Part 1: Validate Skill Routers

### Step 1: Scan All Skill Files

Find all SKILL.md files and extract:
1. **Routers**: Files that route to sub-skills (have routing tables)
2. **Sub-skills**: Files referenced by routers

```bash
LOCAL_BASE=".claude/skills/waramity"

echo "=== Scanning skill files ==="
find "$LOCAL_BASE" -name "SKILL.md" -type f | while read file; do
  echo "Found: ${file#$LOCAL_BASE/}"
done
```

---

### Step 2: Extract References

For each router file, extract:
- Sub-skill names from routing tables
- File paths referenced in the router

```bash
# Extract paths from router files (look for patterns like → `path/SKILL.md`)
grep -E "→.*SKILL\.md" "$file" | sed 's/.*→ *`\{0,1\}\([^`]*SKILL\.md\).*/\1/'
```

---

### Step 3: Validate Each Reference

For each reference found:

1. **Check path exists**: Does the file at the referenced path exist?
2. **Check name matches**: Does the sub-skill's `name:` field match what the router calls it?
3. **Check reverse reference**: Does the sub-skill know it belongs to this router?

---

### Step 4: Generate Validation Report

Output a report in this format:

```
=== Skill Integrity Report ===

✗ BROKEN PATH
  Router: waramity/SKILL.md
  References: .claude/skills/waramity/init/SKILL.md
  Actual: .claude/skills/waramity/setup/init/SKILL.md
  Fix: Update router path

✗ NAME MISMATCH
  Router: setup/SKILL.md
  Calls it: "update"
  File name: "sync" (in sync/SKILL.md frontmatter)
  Fix: Rename sub-skill or update router

✓ OK: dev/planner/SKILL.md
✓ OK: dev/doer/SKILL.md

=== Summary ===
Total skills: 15
Valid: 13
Need update: 2
```

---

### Step 5: Fix Issues (if requested)

When user approves fixes:

#### Fix Broken Path
```bash
# Update the router file with correct path
# Old: → `.claude/skills/waramity/init/SKILL.md`
# New: → `.claude/skills/waramity/setup/init/SKILL.md`
```

#### Fix Name Mismatch
```bash
# Option A: Update router to use new name
# Option B: Update sub-skill frontmatter to use old name
# Always ask user which option they prefer
```

---

## Part 2: Sync to GitHub README

### Step 1: Extract Skill Information

For each SKILL.md, extract:
- `name:` from frontmatter
- `description:` from frontmatter
- Relative path from waramity/

```bash
# Extract name and description from a SKILL.md file
extract_frontmatter() {
  local file="$1"
  awk '/^---$/{n++; next} n==1{print}' "$file" | grep -E "^(name|description):"
}
```

---

### Step 2: Build Skills Table

Generate markdown table with columns:
| Skill | Category | Description |

Categories are derived from the folder structure:
- `setup/init` → Category: setup
- `dev/planner` → Category: dev
- `business/gh-remote` → Category: business

---

### Step 3: Fetch Current README

Use GitHub CLI to get current README content:

```bash
REPO="waramity/waramity-skills"

# Check if README exists
gh api repos/$REPO/contents/README.md --jq '.content' | base64 -d 2>/dev/null
```

---

### Step 4: Update Skills Section

The README should have a marked section for the skills table:

```markdown
<!-- SKILLS-START -->
| Skill | Category | Description |
|-------|----------|-------------|
| init | setup | Initialize waramity skills configuration |
| planner | dev | Analyze and plan implementation tasks |
...
<!-- SKILLS-END -->
```

Replace content between markers with the new table.

---

### Step 5: Push to GitHub

After user confirmation, push the updated README:

```bash
REPO="waramity/waramity-skills"

# Create or update README.md
gh api repos/$REPO/contents/README.md \
  -X PUT \
  -f message="docs: sync skills list from local" \
  -f content="$(echo "$NEW_CONTENT" | base64)" \
  -f sha="$CURRENT_SHA"  # Required for updates, omit for new file
```

---

## Generated Table Format

```markdown
# waramity-skills

Claude Code skills for development workflows.

<!-- SKILLS-START -->
## Skills

| Skill | Category | Description |
|-------|----------|-------------|
| waramity | router | Unified requirement workflow router |
| init | setup | Initialize waramity skills configuration |
| update | setup | Validate routers and sync to GitHub |
| planner | dev | Analyze and plan tasks |
| doer | dev | Implement planned requirements |
| save-wip | dev | Commit unfinished work |
| shipper | dev | Commit and push completed work |
| fail | dev | Revert and delete failed requirements |
| track | dev | Show uncommitted files and commit plan |
| gh-remote | business | GitHub operations without local git |
<!-- SKILLS-END -->

## Installation

...
```

---

## Quick Commands

### Validate only (no GitHub sync)
```
/waramity → "check skills" or "validate skills"
```

### Validate and fix
```
/waramity → "fix skill paths" or "update skill references"
```

### Full update (validate + sync to GitHub)
```
/waramity → "update skills readme" or "sync skills to github"
```

---

## Validation Rules

### Path Validation
```
Router says: `.claude/skills/waramity/foo/SKILL.md`
Check: Does this file exist?
If not: Find similar path (e.g., setup/foo/SKILL.md) and suggest fix
```

### Name Validation
```
Router table has: | **planner** | ... |
Sub-skill has: name: planner
Check: Do these match?
```

### Structure Validation
```
Main router (SKILL.md) → Category routers (setup/SKILL.md, dev/SKILL.md)
Category router → Sub-skills (setup/init/SKILL.md, dev/planner/SKILL.md)
Check: Is the hierarchy consistent?
```

---

## Rules

- Always show full validation report before making changes
- Never auto-fix without user confirmation
- Prefer updating routers over renaming folders
- Keep backup of files before modification
- Show diff of proposed changes
- Ask confirmation before any GitHub operation
- Create README.md if it doesn't exist in the repo
- Preserve content outside the `<!-- SKILLS-START/END -->` markers
- Sort skills alphabetically within each category

---

## Expected Output

```
=== Part 1: Skill Validation ===

Scanning .claude/skills/waramity/...

✓ OK: setup/init/SKILL.md
✓ OK: setup/update/SKILL.md
✓ OK: dev/planner/SKILL.md
✓ OK: dev/doer/SKILL.md
✗ BROKEN PATH: dev/SKILL.md references "dev/tracker/SKILL.md" (not found)
  Suggestion: Change to "dev/track/SKILL.md"

Summary: 11/12 valid, 1 issue found

Fix this issue? [Y/n]

=== Part 2: GitHub README Sync ===

Found 12 skills in .claude/skills/waramity/

Category: setup (2 skills)
  - init: Initialize waramity skills configuration
  - update: Validate routers and sync to GitHub

Category: dev (6 skills)
  - doer: Implement planned requirements
  - fail: Revert and delete failed requirements
  - planner: Analyze and plan tasks
  - save-wip: Commit unfinished work
  - shipper: Commit and push completed work
  - track: Show uncommitted files

Category: business (1 skill)
  - gh-remote: GitHub operations without local git

=== Preview README Update ===

[Shows diff of changes]

Push to waramity/waramity-skills? [Y/n]
```
