---
name: doer
description: Execute a planned requirement from .waramity/dev/requirement/. Reads the REQ, implements the code changes, verifies each acceptance criterion, marks criteria as done, and updates the REQ status to Wait to Review. Trigger on phrases like "do REQ", "implement REQ", "execute REQ", "work on REQ", "do requirement", or when the user references a specific REQ number (e.g. "REQ-0001") and wants it implemented. Also trigger when the user selects a requirement title and says "do this" or "implement this".
---

# Requirement Doer

When the user asks to implement a requirement from `.waramity/dev/requirement/` — read the plan, implement the code changes, verify acceptance criteria, and mark it done. Follow the plan precisely.

## Workflow

### Step 1: Identify the Requirement

1. List both `.waramity/dev/requirement/` and `.waramity/dev/wip/` to see all REQ files (filenames show status emoji: 🟡 Planned, 🚧 WIP, ⏳ Wait to Review, ✅ Done)
2. Find the target file by:
   - Explicit REQ number from user (e.g. "REQ-0001" → file with `0001-` in name)
   - User's selection or description matching a filename title
   - If ambiguous, show the list and ask the user which REQ to implement
3. Read the matched individual file (e.g. `.waramity/dev/requirement/⏳ 0005-auto-smooth-paint-mask-edges.md` or `.waramity/dev/wip/🚧 0003-fix-some-bug.md`)
4. Confirm the REQ has status `🟡 Planned` or `🚧 WIP` — if already `⏳ Wait to Review`, tell the user it's done and ready to ship; stop

### Step 2: Understand the Requirement

Read and internalize:
- **Description**: What needs to change and why
- **Acceptance Criteria**: Each checkbox is a deliverable — all must pass
- **Decisions Made**: These are constraints — follow them, don't re-decide
- **Notes**: Implementation hints, file locations, gotchas

Before coding, read all files mentioned in the Notes section and any files relevant to understanding the change.

### Step 3: Implement

Make the code changes needed to satisfy all acceptance criteria.

**Rules:**
- Follow the decisions from the plan — don't deviate unless something is clearly wrong
- Use file paths and line references from the Notes as starting points, but verify they're still accurate
- Make minimal, focused changes — don't refactor beyond what the REQ asks for
- If a decision from the plan turns out to be wrong or impossible, stop and tell the user

### Step 4: Verify Acceptance Criteria

Go through **each** acceptance criterion one by one:

1. Read the relevant code to confirm the criterion is met
2. Trace the logic to verify correctness (e.g., if criterion says "each shot picks a new target", read the function and confirm it selects fresh each call)
3. Check edge cases mentioned in the Notes

**Do not skip verification.** Each criterion must be individually confirmed with evidence from the code.

### Step 5: Update the Requirement File

After all criteria are verified, edit the individual REQ file in-place and rename the file:

**NEVER delete or remove any file under `.waramity/dev/requirement/` or `.waramity/dev/wip/`. Only rename or edit in-place.**

1. Check off each acceptance criterion: `- [ ]` becomes `- [x]`
2. Update status: `🟡 Planned` becomes `⏳ Wait to Review`
3. Rename the file to reflect new status:
   - If status is now `⏳ Wait to Review`:
     ```bash
     OLD_FILE=$(ls .waramity/dev/requirement/*${REQ_NUMBER}-*.md 2>/dev/null | head -1)
     mv "$OLD_FILE" ".waramity/dev/requirement/⏳ ${REQ_NUMBER}-${SLUG}.md"
     ```
   - If status is `🚧 WIP` (user requests mid-implementation):
     ```bash
     OLD_FILE=$(ls .waramity/dev/requirement/*${REQ_NUMBER}-*.md 2>/dev/null | head -1)
     mv "$OLD_FILE" ".waramity/dev/requirement/🚧 ${REQ_NUMBER}-${SLUG}.md"
     ```
4. **Verify the rename succeeded** — run `ls .waramity/dev/requirement/` and confirm the file now has the emoji prefix. If the rename failed, retry it before proceeding.
5. **Leave WIP files alone** — do NOT delete or move files from `.waramity/dev/wip/`. They stay as-is.

### Step 6: Report

Tell the user:
- What was changed (files and brief description)
- How each acceptance criterion was verified
- Any edge cases or things to watch out for

## Edge Cases

- **REQ references outdated line numbers**: Use the Notes as hints but search for the actual code by function/variable name
- **Code has changed since plan was written**: Adapt the implementation to current code state, but follow the same intent
- **An acceptance criterion can't be met**: Stop, explain why, and ask the user how to proceed
- **Multiple REQs requested**: Implement them one at a time, in order
- **WIP file exists for the REQ**: Leave it untouched — never delete or move files from `.waramity/dev/wip/`
