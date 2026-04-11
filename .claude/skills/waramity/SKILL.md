---
name: waramity
description: Unified requirement workflow skill. Routes to the right sub-skill based on user intent: planner (analyze & plan a task), doer (implement a planned REQ), save-wip (commit unfinished work), or shipper (commit & push a completed REQ). Trigger on phrases like "plan this", "analyze this", "do REQ", "implement REQ", "REQ-NNNN", "save my progress", "push WIP", "checkpoint", "ship REQ", "commit and push", or any mention of .waramity or requirement.md.
---

# Waramity

Identify the user's intent and run the matching sub-skill workflow.

## Routing

Read the user's message and select **one** sub-skill:

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Plan a new task | **planner** | "plan this", "analyze this", "what do I need", "let's plan", "review before coding", code/spec pasted, mentions of ".waramity" or "requirement.md" |
| Implement a planned REQ | **doer** | "do REQ", "implement REQ", "execute REQ", "work on REQ", REQ-NNNN number, "do this" with a REQ selected |
| Save unfinished work | **save-wip** | "save my progress", "checkpoint this", "push WIP", "save even though it's broken", "commit unfinished work", "save draft" |
| Ship a completed REQ | **shipper** | "ship REQ", "commit REQ", "push REQ", "done with REQ", "close REQ", "finish REQ", "commit and push" |

Once identified, follow the full workflow from that sub-skill exactly as if it had been invoked directly:

- **planner** → `.claude/skills/waramity/planner/SKILL.md`
- **doer** → `.claude/skills/waramity/doer/SKILL.md`
- **save-wip** → `.claude/skills/waramity/save-wip/SKILL.md`
- **shipper** → `.claude/skills/waramity/shipper/SKILL.md`

If the intent is ambiguous, ask the user:
> "Which workflow do you want? Plan a new requirement, implement an existing one, save WIP, or ship a completed REQ?"
