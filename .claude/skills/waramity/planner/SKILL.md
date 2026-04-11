---
name: planner
description: Analyze code or a coding task, clarify requirements interactively, and produce a structured plan saved to .waramity/requirement.md. Use this skill whenever the user shares code, a feature idea, bug report, refactor plan, or any development task and wants to plan before coding. Trigger on phrases like "plan this", "check this code", "analyze this requirement", "what do I need", "let's plan", "review before coding", or any time the user pastes code or a spec and wants structured planning. Also trigger when the user mentions ".waramity", "requirement plan", or "requirement.md". This skill asks clarifying questions with 4 options (pre-selecting a reasonable default), writes an issue-style document with a 1–5 difficulty rating (reason + cautions), and appends everything to .waramity/requirement.md so plans accumulate over time.
---

# Requirement Planner

When the user provides code, a task, or a requirement — analyze it, clarify what's actually needed, and produce a structured plan. Do NOT start coding.

## Workflow

### Step 1: Analyze the Input

Read and understand the code or task. Identify:
- What the code/task is about
- What's unclear or ambiguous
- What decisions need to be made
- Potential approaches

### Step 2: Clarify Requirements (Interactive)

Ask the user **1–3 focused questions** using the `ask_user_input_v0` tool. Each question must have **exactly 4 options**.

**Rules for questions:**
- Pre-select the most reasonable option by marking it with `✅` prefix in the option label
- Keep options short and clear
- Cover the most impactful decisions (scope, approach, priority, complexity)
- Don't ask obvious questions — infer what you can from context

**Example question categories** (pick what's relevant, not all):
- Scope: "What scope fits this task?" → `"✅ Minimal fix only"`, `"Fix + add tests"`, `"Fix + refactor surrounding code"`, `"Full rewrite of module"`
- Priority: "How urgent is this?" → `"Low — backlog"`, `"✅ Medium — this sprint"`, `"High — blocking"`, `"Critical — production down"`
- Approach: "Which approach?" → specific technical options relevant to the task
- Target: "What's the goal?" → task-specific outcomes

### Step 3: Write the Plan

After the user answers (or accepts defaults), produce the full plan with these sections:

```markdown
---

## 📋 REQ-{NNNN}: {Title}

**Date:** {YYYY-MM-DD}
**Status:** 🟡 Planned

### Description
{2-4 sentences describing what needs to be done and why, written like a clear issue description}

### User Stories
- **As a** {type of user}, **I want to** {goal} **so that** {benefit}
- {Additional user stories as needed}

### Acceptance Criteria
- [ ] {criterion 1}
- [ ] {criterion 2}
- [ ] {criterion 3}

### Decisions Made
| Question | Choice |
|----------|--------|
| {question 1} | {chosen option} |
| {question 2} | {chosen option} |

### Difficulty

**Level:** {1 / 2 / 3 / 4 / 5} — {label}

**Why this difficulty:**
{2-3 sentences explaining what makes this task easy or hard — e.g., number of files touched, clarity of requirements, risk of regression, design decisions required, domain knowledge needed}

**Cautions:**
- {caution 1 — specific risk or thing to watch out for}
- {caution 2}
- {caution 3 — omit if fewer apply}

### Notes
{Any additional context, warnings, or suggestions}
```

#### Difficulty Guide

| Level | Label | Criteria |
|-------|-------|----------|
| **1** | Trivial | Single-file change, < 20 lines, zero design decisions, no risk of regression. Copy-paste or rename level. |
| **2** | Simple | 1–2 files, < 50 lines, clear requirements, minimal side-effects. Straightforward to implement and verify. |
| **3** | Moderate | 2–5 files, 50–200 lines, some design choices, possible edge cases. Requires reading existing code carefully. |
| **4** | Hard | 5+ files or 200+ lines, architectural decisions, cross-system impact, unclear requirements, or non-trivial debugging. Risk of introducing regressions. |
| **5** | Expert | Deep system knowledge required, security/correctness-critical, ambiguous problem space, large-scale refactor, or significant risk of breaking unrelated behavior. Needs extended thinking and careful planning. |

**Common caution categories** (pick what's relevant):
- Breaking change: touches public API / shared state / DB schema
- Regression risk: logic change in hot path or widely-used utility
- Domain knowledge gap: requires understanding of game physics, network protocol, etc.
- Ambiguous spec: requirements are underspecified — clarify before coding
- Performance sensitive: change affects render loop, frame rate, or real-time logic
- State complexity: interacts with multiple stateful systems simultaneously
- Test coverage gap: area has no tests — manual verification required

### Step 4: Save to File

Save the plan as an individual file inside `.waramity/requirement/`.

Each REQ gets its own file named `{NNNN}-{slug}.md` — never append to a shared file.

**Always use this exact bash pattern — no exceptions:**

```bash
mkdir -p .waramity/requirement

# Find next REQ number by scanning existing filenames
LAST=$(ls .waramity/requirement/ 2>/dev/null | grep -oP '^\d+' | sort -n | tail -1)
NEXT=$(printf "%04d" $((10#${LAST:-0} + 1)))

# Slug the title: lowercase, hyphens, no special chars
SLUG=$(echo "{Title}" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-\|-$//g')

FILE=".waramity/requirement/${NEXT}-${SLUG}.md"

# Write the new REQ file (single > is correct — this is a new file each time)
cat > "$FILE" << 'PLAN'
## 📋 REQ-{NEXT}: {Title}

**Date:** {YYYY-MM-DD}
**Status:** 🟡 Planned
... (full plan content here)
PLAN
```

The file starts directly with `## 📋 REQ-{NEXT}:` — no outer header, no `---` divider at the top.

### Step 5: Present Result

After saving, tell the user:
- Show the plan inline (the full formatted plan)
- Confirm it was saved to `.waramity/requirement/{NNNN}-{slug}.md`
- Ask: "Ready to start working on this, or want to adjust anything?"

## Edge Cases

- **Vague input**: Ask more clarifying questions. Default to Opus + Plan mode + Extended Thinking.
- **Multiple tasks**: Break into separate REQ entries, each with its own config.
- **Just code pasted, no context**: Analyze the code, infer the likely task (fix bug? add feature? refactor?), and ask to confirm.
- **User says "use defaults"**: Accept all ✅ pre-selected options without asking.
