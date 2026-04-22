---
name: connect
description: >
  Connect waramity skills to external skills for cross-referencing.
  Creates connection mappings in .waramity/memory/connected-skill/.
  Triggers on: "connect skill", "link skill", "integrate skill", "connect to",
  "link waramity to", "use external skill", "connect design to ui-ux-pro-max".
---

# Skill: connect

Connect waramity skills to external skills, creating documented integration points.

## Purpose

When a waramity skill (e.g., `design/`) can benefit from an external skill's capabilities (e.g., `ui-ux-pro-max/`), this skill creates a connection file that documents:
- How the skills complement each other
- When to use each skill
- How to chain them together

---

## Workflow

### Step 1: Identify Skills to Connect

Parse user request to identify:
- **Source**: Waramity skill (e.g., `design`, `business`, `dev`)
- **Target**: External skill path (e.g., `.claude/skills/ios-skills/ui-ux-pro-max/`)

If not provided, ask:
> "Which skills do you want to connect?
> - **Waramity skill**: design, business, dev, setup?
> - **External skill path**: (full path to external skill folder)"

---

### Step 2: Read Both Skills

Read the SKILL.md files from both skills:

```bash
# Read waramity skill
cat .claude/skills/waramity/{waramity_skill}/SKILL.md

# Read external skill
cat {external_skill_path}/SKILL.md
```

---

### Step 3: Analyze Connection Points

Identify how the skills complement each other:

| Aspect | Waramity Skill | External Skill |
|--------|----------------|----------------|
| **Purpose** | What it does | What it does |
| **Outputs** | What it creates | What it creates |
| **Inputs** | What it needs | What it needs |
| **Overlap** | Common capabilities | Common capabilities |
| **Unique** | Only this skill | Only this skill |

Determine connection type:
- **Extension**: External skill extends waramity capability
- **Complement**: Skills work together on different aspects
- **Override**: External skill replaces waramity sub-skill
- **Reference**: External skill provides data/rules for waramity

---

### Step 4: Create Connection Directory

```bash
mkdir -p .waramity/memory/connected-skill
```

---

### Step 5: Create Connection File

Create file: `.waramity/memory/connected-skill/{waramity-skill}--{external-skill}.md`

**File naming convention:**
- Use skill names only (not full paths)
- Separate with double dash `--`
- All lowercase
- Examples:
  - `design--ui-ux-pro-max.md`
  - `business--market-research.md`
  - `dev--swift-ios-skills.md`

**File template:**

```markdown
# Connection: {waramity-skill} <-> {external-skill}

## Overview

**Waramity Skill**: {waramity-skill}
**External Skill**: {external-skill}
**Connection Type**: {Extension|Complement|Override|Reference}

## How They Connect

{Explanation of how the skills work together}

## When to Use Each

### Use {waramity-skill} when:
- {condition 1}
- {condition 2}

### Use {external-skill} when:
- {condition 1}
- {condition 2}

### Use Both Together when:
- {condition 1}
- {condition 2}

## Integration Workflow

1. **Start with**: {skill}
2. **Then use**: {skill}
3. **Output goes to**: {location}

## Example Chain

**User Request**: "{example request}"

1. {waramity-skill} → {action} → {output}
2. {external-skill} → {action} → {output}
3. Combined result → {final output}

## Cross-Reference Commands

| Action | Command |
|--------|---------|
| {action 1} | `/waramity` → "{trigger phrase}" |
| {action 2} | {external skill command} |

## Shared Outputs

| Output | Used By | Location |
|--------|---------|----------|
| {output 1} | Both | {path} |
| {output 2} | {skill} | {path} |

---

*Connected on: {date}*
*Last updated: {date}*
```

---

### Step 6: Confirm Connection

Output summary:
```
Connection created!

File: .waramity/memory/connected-skill/{waramity-skill}--{external-skill}.md

{waramity-skill} → {external-skill}
Type: {connection type}

Integration points:
- {point 1}
- {point 2}
```

---

## Example: Connecting design to ui-ux-pro-max

**User**: "connect design to ui-ux-pro-max"

1. **Source**: `.claude/skills/waramity/design/`
2. **Target**: `.claude/skills/ios-skills/ui-ux-pro-max/`

**Connection Analysis**:
- design/brand-kit → ui-ux-pro-max colors/typography
- design/ui-mockup → ui-ux-pro-max styles/stacks
- design/ux-flow → ui-ux-pro-max ux-guidelines

**Output File**: `.waramity/memory/connected-skill/design--ui-ux-pro-max.md`

**Connection Type**: Extension (ui-ux-pro-max extends design with rich database)

---

## Rules

- Always read both SKILL.md files before creating connection
- Create directory if it doesn't exist
- Use consistent file naming: `{source}--{target}.md`
- Document specific integration points, not vague descriptions
- Include example workflows with actual commands
- Update connection file if either skill changes

---

## Quick Commands

### Connect waramity skill to external skill
```
/waramity → "connect design to ui-ux-pro-max"
```

### List existing connections
```bash
ls -la .waramity/memory/connected-skill/
```

### Read a connection
```bash
cat .waramity/memory/connected-skill/design--ui-ux-pro-max.md
```

### Remove a connection
```bash
rm .waramity/memory/connected-skill/design--ui-ux-pro-max.md
```
