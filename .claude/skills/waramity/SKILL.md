---
name: waramity
description: Unified requirement workflow skill. Routes to the right sub-skill based on user intent: planner (analyze & plan a task), doer (implement a planned REQ), save-wip (commit unfinished work), shipper (commit & push a completed REQ), fail (revert code and delete a REQ/WIP), track (show uncommitted files and commit plan), or gh-remote (GitHub operations without local git). Trigger on phrases like "plan this", "analyze this", "do REQ", "implement REQ", "REQ-NNNN", "save my progress", "push WIP", "checkpoint", "ship REQ", "commit and push", "fail REQ", "revert REQ", "abandon REQ", "track changes", "what should I commit", or any mention of .waramity or requirement.md.
---

# Waramity

Identify the user's intent and run the matching sub-skill workflow.

## Routing

Read the user's message and select **one** sub-skill:

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Initialize project | **init** | "init waramity", "setup waramity", "initialize .waramity", "pull latest skill", "update waramity skill", "setup project" |
| Plan a new task | **planner** | "plan this", "analyze this", "what do I need", "let's plan", "review before coding", code/spec pasted, mentions of ".waramity" or "requirement.md" |
| Implement a planned REQ | **doer** | "do REQ", "implement REQ", "execute REQ", "work on REQ", REQ-NNNN number, "do this" with a REQ selected |
| Save unfinished work | **save-wip** | "save my progress", "checkpoint this", "push WIP", "save even though it's broken", "commit unfinished work", "save draft" |
| Ship a completed REQ | **shipper** | "ship REQ", "commit REQ", "push REQ", "done with REQ", "close REQ", "finish REQ", "commit and push" |
| Track uncommitted files | **track** | "track changes", "what should I commit", "show uncommitted", "commit list", "pending changes", "what's changed" |
| GitHub remote operations | **gh-remote** | "do on remote", "gh remote", "gh-remote", "operate remotely", "no local", "via gh", PR/branch number with remote-only intent |
| Abandon a REQ | **fail** | "fail REQ", "revert REQ", "abandon REQ", "scrap REQ", "undo REQ", "cancel REQ", "throw away REQ", "discard REQ" |

Once identified, follow the full workflow from that sub-skill exactly as if it had been invoked directly:

- **init** → `.claude/skills/waramity/init/SKILL.md`
- **planner** → `.claude/skills/waramity/planner/SKILL.md`
- **doer** → `.claude/skills/waramity/doer/SKILL.md`
- **save-wip** → `.claude/skills/waramity/save-wip/SKILL.md`
- **shipper** → `.claude/skills/waramity/shipper/SKILL.md`
- **track** → `.claude/skills/waramity/track/SKILL.md`
- **gh-remote** → `.claude/skills/waramity/gh-remote/SKILL.md`
- **fail** → `.claude/skills/waramity/fail/SKILL.md`

If the intent is ambiguous, ask the user:
> "Which workflow do you want? Initialize project, plan a new requirement, implement an existing one, save WIP, ship a completed REQ, track changes, operate on remote, or abandon (fail) a REQ?"
