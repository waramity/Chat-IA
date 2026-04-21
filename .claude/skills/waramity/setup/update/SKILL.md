---
name: update
description: >
  Validate and update skill references across router files.
  Checks for broken paths, outdated names, and mismatched connections.
  Triggers on: "check skills", "validate skills", "fix skill paths",
  "skill integrity", "update skill references", "check skill routes".
---

# Skill: update

Validate skill router references and fix broken/outdated connections.

## Purpose

Scans all skill files to find:
- Router files that reference non-existent paths
- Sub-skills with names that don't match router entries
- Broken connections between parent routers and sub-skills

---

## Workflow

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

### Step 4: Generate Report

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

### Step 5: Ask User

Present the issues found and ask:

> "Found {N} issues in skill references:
>
> 1. **{issue_type}**: {description}
>    - Current: {current_value}
>    - Should be: {expected_value}
>
> Do you want me to fix these? (yes/no/select specific)"

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

## Fix Actions

When user approves fixes:

### Fix Broken Path
```bash
# Update the router file with correct path
# Old: → `.claude/skills/waramity/init/SKILL.md`
# New: → `.claude/skills/waramity/setup/init/SKILL.md`
```

### Fix Name Mismatch
```bash
# Option A: Update router to use new name
# Option B: Update sub-skill frontmatter to use old name
# Always ask user which option they prefer
```

### Fix Missing Reference
```bash
# Add missing sub-skill to router table
# Or remove orphan sub-skill if not needed
```

---

## Quick Commands

### Check only (no changes)
```
/waramity → "check skills" or "validate skills"
```

### Check and fix
```
/waramity → "fix skill paths" or "update skill references"
```

---

## Expected Output Example

```
Scanning .claude/skills/waramity/...

=== Issues Found ===

1. BROKEN PATH in waramity/SKILL.md:47
   - References: .claude/skills/waramity/init/SKILL.md
   - File not found at this path
   - Suggestion: Change to .claude/skills/waramity/setup/init/SKILL.md

2. BROKEN PATH in waramity/SKILL.md:48
   - References: .claude/skills/waramity/update/SKILL.md
   - File not found at this path
   - Suggestion: Change to .claude/skills/waramity/setup/sync/SKILL.md

3. NAME MISMATCH in setup/SKILL.md:19
   - Router calls it: "update"
   - Sub-skill name: "sync" (from setup/sync/SKILL.md)
   - Suggestion: Update router or rename folder back to "update"

=== Summary ===
Files scanned: 18
Issues found: 3

Update these references? [Y/n/select]
```

---

## Rules

- Always show full report before making changes
- Never auto-fix without user confirmation
- Prefer updating routers over renaming folders
- Keep backup of files before modification
- Show diff of proposed changes
