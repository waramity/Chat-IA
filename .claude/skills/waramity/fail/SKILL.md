---
name: fail
description: Abandon a requirement by reverting only the code changes related to it, then removing its REQ file and any associated WIP file. Trigger on phrases like "fail REQ", "revert REQ", "abandon REQ", "scrap REQ", "undo REQ", "cancel REQ", "throw away REQ", or when the user wants to discard work tied to a specific requirement.
---

# Requirement Fail

Revert code changes tied to a specific REQ, then delete its REQ file and any associated WIP file. Only touches files explicitly linked to the REQ — leaves everything else untouched.

## Workflow

### Step 1: Identify the REQ to Fail

1. List `.waramity/requirement/` and `.waramity/wip/`
2. Find the target REQ by explicit number or description match. If ambiguous, show the list and ask.
3. Read the REQ file to understand scope
4. Read the WIP file if one exists — the "Files Changed" section is the primary source of truth for what to revert

### Step 2: Find REQ Commits in Git History

```bash
git log --oneline --all | grep -i "{NNNN}"
```

Show the user the list of commits found. If none found, all changes are uncommitted — proceed to Step 3.

### Step 3: Build the Revert File List

Collect files from **exactly two sources** — nothing more:

**Source A — Each REQ commit (one command per commit hash from Step 2):**
```bash
git diff-tree --no-commit-id -r --name-only {commit-hash}
```
Run this for every commit found. Collect all unique paths.

**Source B — WIP file "Files Changed" section** (if WIP exists from Step 1).

Merge both lists, deduplicate. Show the user:

> "These files will be reverted as part of {REQ-ID}:"
>
> - `file/a.js`
> - `file/b.js`
>
> "Confirm? Any files to exclude?"

Wait for user confirmation. Store confirmed list as `{REVERT-FILES}`.

### Step 4: Determine Revert Base

```bash
EARLIEST=$(git log --all --oneline | grep -i "{NNNN}" | tail -1 | awk '{print $1}')
BASE_COMMIT="${EARLIEST}^"
```

If no REQ commits exist (all changes are uncommitted), `BASE_COMMIT` is `HEAD`.

### Step 5: Revert the Files

For each file in `{REVERT-FILES}`:

```bash
# Check if the file existed at base
git show {BASE_COMMIT}:{file} 2>/dev/null && echo "EXISTS" || echo "NEW_FILE"
```

- If **EXISTS** → restore it: `git checkout {BASE_COMMIT} -- {file}`
- If **NEW_FILE** (didn't exist before the REQ) → delete it: `git rm {file}`

When `BASE_COMMIT` is `HEAD` (no commits):
- Restore modified files: `git checkout HEAD -- {file}`
- Delete newly created files: `rm {file}`

After all files are restored, run `rtk git diff --staged` and show the user the summary.

### Step 6: Remove the REQ and All Related WIP Files

**Rule: removing the REQ always removes every WIP file that shares its number — no exceptions.**

```bash
REQ_FILE=$(ls ".waramity/requirement/"*"{NNNN}"*.md 2>/dev/null | head -1)

# Find ALL WIP files with this REQ number (there may be more than one)
WIP_FILES=$(ls ".waramity/wip/"*"{NNNN}"*.md 2>/dev/null)

git rm "$REQ_FILE"

for WIP in $WIP_FILES; do
  git rm "$WIP"
done
```

If no WIP files are found that is fine — continue. If the REQ file is not found, warn the user and stop.

### Step 7: Commit Everything

Stage the reverted code files:
```bash
rtk git add {REVERT-FILES}
```

Commit with format:
```
Revert REQ-{NNNN}: {REQ Title}

Reverted files: {comma-separated list}
Reason: Requirement abandoned — changes discarded

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 8: Push

```bash
rtk git push
```

If push fails: `rtk git pull --rebase` then `rtk git push`. Never force push without explicit user approval.

### Step 9: Report

Tell the user:
- Commit hash and branch
- Which files were reverted and to what base
- Which REQ file was deleted
- Which WIP file was deleted (if any)
- How many REQ files remain in `.waramity/requirement/`

## Edge Cases

- **No commits found for REQ**: All changes are uncommitted. Restore with `git checkout HEAD -- {file}`. For untracked new files, use `rm {file}`.
- **REQ commits interleaved with unrelated commits**: Do NOT use `git revert`. Restore each file individually with `git checkout {BASE_COMMIT} -- {file}`.
- **File also modified by a later REQ**: Warn the user — reverting to before this REQ may undo the later REQ's work too. Show the conflict and wait for resolution before continuing.
- **User wants to exclude some files**: Remove them from `{REVERT-FILES}` and skip during Step 5.
- **REQ file doesn't exist**: Warn the user. Ask if they still want to revert the code changes manually.
- **Push rejected**: Run `rtk git pull --rebase` then retry.
