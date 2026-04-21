---
name: save-wip
description: Save, commit, and push work-in-progress code that may have bugs or is not yet complete. Asks whether to push to the same branch or a new branch, records a WIP label explaining what's unfinished and why, and appends a checkpoint entry to .waramity/wip.md for future reference. Trigger on phrases like "save my progress", "checkpoint this", "push WIP", "save even though it's broken", "commit unfinished work", "save draft", "push wip", "checkpoint", or when the user is unsure if the code works but wants to preserve and push it.
---

# Requirement Save WIP

Push work-in-progress code that may be broken or incomplete. Records what was done, what is unfinished, and why — so nothing is lost and the status is always clear.

## Workflow

### Step 1: Gather Context

1. Run `rtk git status` to see all changed files
2. Run `rtk git diff` to review the changes
3. Note the current branch name: `git branch --show-current`

### Step 2: Ask the User — Branch & WIP Details

Ask the user **3 focused questions** using the `ask_user_input_v0` tool:

**Question 1 — Branch target:**
> "Where should this WIP commit go?"
- `"✅ Same branch (current: {current-branch})"`
- `"New branch from current"`
- `"New branch from main"`
- `"New branch from another base"`

**Question 2 — WIP status label:**
> "What best describes why this isn't finished?"
- `"✅ Has bugs / broken behavior"`
- `"Incomplete feature — partially done"`
- `"Needs testing / unsure if correct"`
- `"Experimental — may be thrown away"`

**Question 3 — What still needs to be done:**
> "What needs to happen before this is considered done?"
- `"✅ I'll describe it in the commit message"`
- `"Nothing — just preserving progress"`
- `"Fix the bug I just described"`
- `"Continue implementing the feature"`

### Step 3: Handle Branch

**If same branch:** continue — no branch change needed.

**If new branch:**
1. Ask for the branch name (suggest: `wip/{short-description}-{YYYY-MM-DD}`)
2. Create and switch: `git checkout -b {branch-name}`
3. If "from main": first `git fetch origin main` then `git checkout -b {branch-name} origin/main`
4. If "from another base": ask which branch to base off

### Step 4: Stage and Commit

1. Stage all changed files: `rtk git add {files}` (prefer explicit file names over `git add .`)
2. Prompt the user: "Briefly describe what you changed (for the commit message):" — use `ask_user_input_v0` if needed, or infer from the diff
3. Create the commit using this format:

**Commit message format:**
```
WIP: {brief description of what was changed}

Status: {WIP label from Step 2}
Unfinished: {what still needs to be done}

Co-Authored-By: Claude <noreply@anthropic.com>
```

**Example:**
```
WIP: Add ball collision response with wall boundaries

Status: Has bugs / broken behavior
Unfinished: Balls sometimes phase through corners — corner case not handled yet

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 5: Push

1. Push to remote: `rtk git push` (add `-u origin {branch-name}` if new branch)
2. If push fails due to upstream changes: `rtk git pull --rebase` then `rtk git push`
3. Capture the commit hash: `git rev-parse --short HEAD`

### Step 5b: Update REQ Status and Capture Path

If the WIP work is tied to a REQ file in `.waramity/requirement/`:

1. Find the relevant REQ file (by matching the work description or files changed)
2. Capture the relative path from `.waramity/wip/` — e.g., `../requirement/0010-fix-ball-bounce-skip-frame.md`
3. If found and status is `🟡 Planned`, update it to `🚧 WIP`

This keeps the requirement status honest — the work has started but is not yet finished.

**Capture the REQ path for use in Step 6:** store it in variable `{REQ-PATH}` (e.g., `../requirement/0010-fix-ball-bounce-skip-frame.md`). If no REQ was found, leave `{REQ-PATH}` empty — the field will be omitted from the WIP file.

### Step 6: Record in .waramity/wip/

Save a checkpoint entry as an individual file inside `.waramity/wip/`.

Each WIP gets its own file named `{NNNN}-{slug}.md` — never append to a shared file.

**Always use this exact bash pattern — no exceptions:**

```bash
mkdir -p .waramity/wip

# Find next WIP number by scanning existing filenames
LAST=$(ls .waramity/wip/ 2>/dev/null | grep -oP '^\d+' | sort -n | tail -1)
NEXT=$(printf "%04d" $((10#${LAST:-0} + 1)))

# Slug the brief description: lowercase, hyphens, no special chars
SLUG=$(echo "{Brief Description}" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-\|-$//g')

FILE=".waramity/wip/${NEXT}-${SLUG}.md"

# Write the new WIP file (single > is correct — this is a new file each time)
cat > "$FILE" << 'ENTRY'
## 🚧 WIP-{NEXT}: {Brief Description}

**Date:** {YYYY-MM-DD}
**Commit:** `{short-hash}` on `{branch-name}`
**Status:** {WIP label}

### What Was Done
{2-4 sentences describing what was changed and what was attempted}

### Why It's Not Finished
{Clear explanation of what's broken, missing, or uncertain}

### What Needs to Happen Next
{Concrete next steps — what must be fixed, implemented, or tested before this is considered done}

### Files Changed
{List of files that were staged in this commit}
ENTRY
```

The file starts directly with `## 🚧 WIP-{NEXT}:` — no outer header, no `---` divider at the top.

### Step 7: Report

Tell the user:
- Commit hash and branch it was pushed to
- The WIP label applied
- Confirm `.waramity/wip/{NNNN}-{slug}.md` was created
- What needs to happen next (echo back what they said)
- Reminder: "When this is fixed and ready to ship, use `/shipper` or just commit normally."

## Edge Cases

- **No changed files:** Tell the user there's nothing to commit. Ask if they want to create an empty checkpoint note in `wip.md` anyway (for tracking purposes).
- **User says "same branch" but current branch is `main` or `master`:** Warn the user — pushing WIP directly to main is risky. Suggest creating a WIP branch instead. Let the user decide.
- **User doesn't know what's broken:** That's fine — use label `"Needs testing / unsure if correct"` and leave "Why It's Not Finished" as `"Behavior is uncertain — needs manual verification"`.
- **Multiple unrelated WIP changes:** Note this in the WIP entry. Suggest the user split into separate commits next time, but don't block them.
- **Push rejected (not fast-forward):** Run `rtk git pull --rebase` then retry push. If rebase fails, stop and tell the user — don't force push without explicit permission.
- **User wants to push force:** Ask for explicit confirmation before running `git push --force-with-lease`. Never use `--force`.
