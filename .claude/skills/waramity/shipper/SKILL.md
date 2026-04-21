---
name: shipper
description: Commit and push code for a completed requirement, then remove it from .waramity/requirement.md. Use after doer has finished implementing a REQ. Trigger on phrases like "ship REQ", "commit REQ", "push REQ", "ship requirement", "commit and push", "done with REQ", "close REQ", "finish REQ", or when the user wants to commit work tied to a specific requirement.
---

# Requirement Shipper

After a requirement has been implemented and marked `⏳ Wait to Review`, this skill commits the changes, pushes to remote, and removes the REQ entry from the plan file.

## Workflow

### Step 1: Identify the Requirement

1. List `.waramity/requirement/` to see all REQ files (filenames show status emoji: 🟡 Planned, 🚧 WIP, ⏳ Wait to Review, ✅ Done)
2. Find the target file by:
   - Explicit REQ number from user (e.g. "REQ-0001" → file with `0001-` in name)
   - User's selection or description matching a filename title
   - If only one REQ file has status `✅ Done`, use that one
   - If ambiguous, show the list and ask the user which REQ to ship
3. Read the matched individual file
4. Confirm the REQ has status `⏳ Wait to Review` — if still `🟡 Planned`, tell the user to implement it first and stop
5. Save the REQ title, description, and filename for the commit message

### Step 2: Stage and Commit

1. Run `rtk git status` to see all changed files
2. Run `rtk git diff` to review the changes
3. Stage the relevant changed files (do NOT stage `.waramity/requirement.md` yet — that gets updated in Step 3)
4. Create a commit with a message derived from the REQ:

**Commit message format:**
```
{Brief description from REQ title/description}

REQ-{NNNN}: {REQ Title}

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Example:**
```
Fix sniper streak targeting to pick independent random targets

REQ-0001: Fix Weapon Streak Targeting

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 3: Remove the REQ File

Find and delete the REQ file by REQ number (avoids emoji/spacing mismatches):

```bash
REQ_FILE=$(ls ".waramity/requirement/"*{NNNN}*.md 2>/dev/null | head -1)
rm "$REQ_FILE"
```

Stage and commit the deletion:

```
Remove completed REQ-{NNNN} from requirement plan
```

### Step 4: Push

1. Push to remote: `rtk git push`
2. If push fails due to upstream changes, run `rtk git pull --rebase` then `rtk git push`

### Step 5: Report

Tell the user:
- Commit hash and message
- Which REQ file was deleted
- Confirm push succeeded
- How many REQ files remain in `.waramity/requirement/`

## Edge Cases

- **No changed files**: If `git status` shows no changes, the code may already be committed. Ask the user if they want to just remove the REQ from the plan.
- **Unrelated changes in working tree**: Only stage files relevant to the REQ. If unsure which files are related, show the diff and ask the user.
- **Multiple done REQs**: Ship them one at a time unless the user asks to batch them.
- **Last REQ file being shipped**: The `.waramity/requirement/` directory will be empty — that's fine, leave the directory.
