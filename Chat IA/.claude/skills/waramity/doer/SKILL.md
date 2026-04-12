---
name: doer
description: Execute a planned requirement from .waramity/requirement.md. Reads the REQ, implements the code changes, verifies each acceptance criterion, marks criteria as done, and updates the REQ status to Done. Trigger on phrases like "do REQ", "implement REQ", "execute REQ", "work on REQ", "do requirement", or when the user references a specific REQ number (e.g. "REQ-0001") and wants it implemented. Also trigger when the user selects a requirement title and says "do this" or "implement this".
---

# Requirement Doer

When the user asks to implement a requirement from `.waramity/requirement.md` — read the plan, implement the code changes, verify acceptance criteria, and mark it done. Follow the plan precisely.

## Workflow

### Step 1: Identify the Requirement

1. List `.waramity/requirement/` to see all REQ files
2. Find the target file by:
   - Explicit REQ number from user (e.g. "REQ-0001" → file starting with `0001-`)
   - User's selection or description matching a filename title
   - If ambiguous, show the list and ask the user which REQ to implement
3. Read the matched individual file (e.g. `.waramity/requirement/0005-auto-smooth-paint-mask-edges.md`)
4. Confirm the REQ has status `🟡 Planned` — if already `✅ Done`, tell the user and stop

### Step 2: Understand the Requirement

Read and internalize:
- **Description**: What needs to change and why
- **Acceptance Criteria**: Each checkbox is a deliverable — all must pass
- **Decisions Made**: These are constraints — follow them, don't re-decide
- **Notes**: Implementation hints, file locations, gotchas

Before coding, read all files mentioned in the Notes section and any files relevant to understanding the change.

Invoke `agent-skills:context-engineering` to ensure the right files and context are loaded before writing any code.

### Step 3: Implement

Make the code changes needed to satisfy all acceptance criteria.

Follow `agent-skills:incremental-implementation` — build one thin vertical slice at a time, verify it works before expanding. Do not write all the code at once.

Apply contextual skills based on what the REQ touches:
- **UI/canvas/rendering work** → invoke `agent-skills:frontend-ui-engineering`
- **New interfaces or APIs** → invoke `agent-skills:api-and-interface-design`
- **Security-sensitive changes** → invoke `agent-skills:security-and-hardening`
- **Render-loop or real-time performance** → invoke `agent-skills:performance-optimization`
- **Something breaks during implementation** → invoke `agent-skills:debugging-and-error-recovery`

**Rules:**
- Follow the decisions from the plan — don't deviate unless something is clearly wrong
- Use file paths and line references from the Notes as starting points, but verify they're still accurate
- Make minimal, focused changes — don't refactor beyond what the REQ asks for
- If a decision from the plan turns out to be wrong or impossible, stop and tell the user

### Step 4: Verify Acceptance Criteria

Invoke `agent-skills:test-driven-development` to prove each criterion. For browser/canvas rendering, use `agent-skills:browser-testing-with-devtools` for runtime verification.

Go through **each** acceptance criterion one by one:

1. Read the relevant code to confirm the criterion is met
2. Trace the logic to verify correctness (e.g., if criterion says "each shot picks a new target", read the function and confirm it selects fresh each call)
3. Check edge cases mentioned in the Notes

**Do not skip verification.** Each criterion must be individually confirmed with evidence from the code.

After all criteria pass, invoke `agent-skills:code-review-and-quality` for a five-axis review (correctness, readability, architecture, security, performance) before marking the REQ done.

### Step 5: Update the Requirement File

After all criteria are verified, edit the individual REQ file in-place:

1. Check off each acceptance criterion: `- [ ]` becomes `- [x]`
2. Update status: `🟡 Planned` becomes `✅ Done`

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
