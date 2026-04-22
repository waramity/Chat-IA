---
name: work-breakdown
description: >
  Generate a Work Breakdown Structure (WBS) with actionable Claude Code prompts.
  Use this skill when the user wants to break down a project, feature, or task into smaller pieces
  that can be executed in separate Claude Code sessions.
  Triggers on: "work breakdown", "WBS", "break this down", "break down tasks", "divide tasks", "split tasks",
  "create prompts for each task", "generate task prompts", "breakdown to prompts",
  "prepare prompts", "Claude Code prompts", "session prompts", "actionable tasks".
  Output is a structured breakdown with copy-ready prompts for each work package.
---

# Work Breakdown Skill

Creates a **Work Breakdown Structure (WBS)** with copy-ready prompts designed for use in separate Claude Code sessions.

---

## Trigger Contexts

Activate this skill when:
- "break down [project/feature] into tasks"
- "create work breakdown for [idea]"
- "break down tasks for me"
- "generate prompts for each task"
- "prepare Claude Code sessions for [project]"
- "split this into actionable pieces"
- User has a large task and wants structured execution plan

---

## Step 1 — Gather Context

Before creating WBS, ensure you have:

| Info | Why Needed |
|------|------------|
| Project/feature description | Core subject to break down |
| Scope boundaries | What's in/out of scope |
| Tech stack (if known) | Affects task specifics |
| Dependencies/constraints | Ordering and blockers |
| Desired granularity | How small should tasks be? (hours/days/sessions) |

If missing critical info, ask concisely. If context is already in conversation, extract directly.

---

## Step 2 — Analyze & Decompose

Break down the work using these principles:

### Decomposition Rules
1. **100% Rule**: Child tasks must cover 100% of parent scope
2. **Mutual Exclusivity**: No overlapping work between siblings
3. **Outcome-Based**: Each task produces a verifiable deliverable
4. **Session-Sized**: Each task should be completable in one Claude Code session (15-60 min ideal)
5. **Self-Contained**: Each prompt should have enough context to execute independently

### Hierarchy Levels
```
Level 1: Project/Epic
  Level 2: Phase/Milestone
    Level 3: Work Package (this is what gets a prompt)
      Level 4: Tasks (optional, for complex packages)
```

---

## Step 3 — Generate WBS Document

Output as a markdown document with this structure:

```markdown
# Work Breakdown Structure: [Project Name]

## Overview
- **Project**: [Name]
- **Scope**: [Brief scope statement]
- **Total Work Packages**: [N]
- **Estimated Sessions**: [N]

---

## Phase 1: [Phase Name]

### WP-1.1: [Work Package Name]
**Deliverable**: [What this produces]
**Dependencies**: [WP-X.X or "None"]
**Verification**: [How to verify completion]

<prompt>
[Context paragraph - what exists, what's needed, constraints]

[Specific task instructions]

[Expected output/deliverable description]

[Any file paths, APIs, or references needed]
</prompt>

---

### WP-1.2: [Next Work Package]
...
```

---

## Step 4 — Prompt Writing Guidelines

Each `<prompt>` block must be:

### Structure
```
[1-2 sentences of context - what exists now]

[Clear task statement - what to do]

[Specifics - files to create/modify, APIs to use, patterns to follow]

[Success criteria - how to verify it's done]

[Optional: constraints or things to avoid]
```

### Quality Checklist
- [ ] Self-contained — doesn't require reading previous conversation
- [ ] Specific — mentions exact files, functions, or components
- [ ] Verifiable — clear success criteria
- [ ] Appropriately scoped — not too big, not trivially small
- [ ] No ambiguous pronouns — use explicit names

### Example Prompt (Good)
```
The project has a Next.js app with Prisma ORM. The User model exists in prisma/schema.prisma.

Add email verification to the signup flow:
1. Add `emailVerified` boolean and `verificationToken` string fields to User model
2. Create /api/auth/verify endpoint that validates token and updates user
3. Modify signup to generate token and send verification email via Resend
4. Add verification pending page at /auth/verify-pending

Use the existing email service in src/lib/email.ts. Follow the error handling pattern in other API routes.

Verify by: running `npx prisma migrate dev`, checking endpoint responds correctly, and testing full flow.
```

### Example Prompt (Bad)
```
Add email verification to the app.
```

---

## Step 5 — Dependencies & Ordering

Include a dependency graph in the output:

```markdown
## Execution Order

### Critical Path
WP-1.1 → WP-1.2 → WP-2.1 → WP-2.3 → WP-3.1

### Parallel Opportunities
- WP-1.3 and WP-1.4 can run in parallel after WP-1.2
- WP-2.2 is independent, can start anytime

### Blockers
- WP-2.1 blocks all of Phase 3
- External: API key needed before WP-2.3
```

---

## Step 6 — Output Format Options

Ask user preference if not specified:

| Format | Best For |
|--------|----------|
| **Single File** | Small projects (≤10 WPs), easy to share |
| **Multi-File** | Large projects, one file per phase |
| **Checklist Mode** | Add `- [ ]` for tracking completion |
| **JSON Export** | Integration with task management tools |

Default to **Single File with Checklist**.

---

## Step 7 — Save & Present

Save to: `.waramity/business/wbs/[project-name]-wbs.md`

Or if user specifies output location, use that.

Present summary:
```
Created WBS with:
- [N] Phases
- [N] Work Packages
- [N] total prompts ready to use

Critical path: [describe]
Recommended start: WP-X.X ([name])
```

---

## Special Modes

### Quick Mode
If user says "quick breakdown" or "rough WBS":
- Skip detailed dependency analysis
- Use shorter prompts (3-4 lines)
- Don't ask clarifying questions

### Detailed Mode
If user says "detailed breakdown" or "comprehensive WBS":
- Add Level 4 tasks within work packages
- Include time estimates per WP
- Add risk notes per phase
- Generate test prompts alongside implementation prompts

### Code-First Mode
If project involves coding, for each WP also generate:
- Test prompt (write tests first)
- Implementation prompt
- Review prompt (optional)

---

## Thai Language Support

If user writes in Thai or project targets Thai market:
- Use Thai for WP names and descriptions
- Keep prompts in English (Claude Code works best in English)
- Add localization context where relevant (e.g., "use Thai language in UI")

---

## Integration with Other Skills

After WBS creation, suggest:
- **planner** → For detailed REQ planning of individual WPs
- **doer** → To execute a specific WP
- **track** → To monitor progress across WPs
