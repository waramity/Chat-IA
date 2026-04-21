---
name: waramity
description: Unified requirement workflow skill. Routes to the right sub-skill based on user intent. Utility skills: init, sync, update. Dev skills: planner, doer, save-wip, shipper, fail, track, gh-remote. Business skills: business-plan, app-analysis, keyword-trend, solo-rank, work-breakdown. Design skills: brand-kit, ux-flow, wireframe, ui-mockup, icon-set, ux-copy. Trigger on utility phrases like "init waramity", "sync skills", "push skills", "pull skills", "check skills", "validate skills". Trigger on dev phrases like "plan this", "do REQ", "save WIP", "ship REQ", "fail REQ", "track changes". Trigger on business phrases like "business plan", "analyze app", "keyword trend", "solo developer app", "work breakdown", "WBS", "break this down". Trigger on design phrases like "brand kit", "ux flow", "wireframe", "ui mockup", "icon set", "ux copy", "color palette", "typography".
---

# Waramity

Identify the user's intent and run the matching sub-skill workflow.

## Routing

Read the user's message and select **one** sub-skill:

### Utility Skills

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Initialize project | **init** | "init waramity", "setup waramity", "initialize .waramity", "setup project" |
| Sync skills local/remote | **sync** | "sync skills", "push skills", "pull skills", "sync waramity", "upload skills to github", "download latest skills", "push local to remote", "pull remote to local" |
| Validate skill references | **update** | "check skills", "validate skills", "fix skill paths", "skill integrity", "update references" |

### Dev Skills

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Plan a new task | **planner** | "plan this", "analyze this", "what do I need", "let's plan", "review before coding", code/spec pasted, mentions of ".waramity" or "requirement.md" |
| Implement a planned REQ | **doer** | "do REQ", "implement REQ", "execute REQ", "work on REQ", REQ-NNNN number, "do this" with a REQ selected |
| Save unfinished work | **save-wip** | "save my progress", "checkpoint this", "push WIP", "save even though it's broken", "commit unfinished work", "save draft" |
| Ship a completed REQ | **shipper** | "ship REQ", "commit REQ", "push REQ", "done with REQ", "close REQ", "finish REQ", "commit and push" |
| Track uncommitted files | **track** | "track changes", "what should I commit", "show uncommitted", "commit list", "pending changes", "what's changed" |
| GitHub remote operations | **gh-remote** | "do on remote", "gh remote", "gh-remote", "operate remotely", "no local", "via gh", PR/branch number with remote-only intent |
| Abandon a REQ | **fail** | "fail REQ", "revert REQ", "abandon REQ", "scrap REQ", "undo REQ", "cancel REQ", "throw away REQ", "discard REQ" |

### Business Skills

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Create business plan | **business-plan** | "business plan", "market analysis", "revenue model", "monetization plan", "go-to-market", "GTM strategy" |
| Analyze single app idea | **app-analysis** | "analyze app idea", "app business plan", "tech stack for app", "MVP roadmap", "app monetization", "help analyze app" |
| Find keyword trends | **keyword-trend** | "keyword trend", "trending keywords", "search trends", "analyze keywords", "popular searches" |
| Rank solo-dev apps | **solo-rank** | "low risk app", "solo developer app", "passive income app", "minimal cost app", "build alone" |
| Create work breakdown | **work-breakdown** | "work breakdown", "WBS", "break this down", "split tasks", "divide work", "create prompts", "session prompts", "actionable tasks" |

### Design Skills

| Intent | Sub-skill | Trigger phrases |
|--------|-----------|-----------------|
| Define brand identity | **brand-kit** | "brand kit", "define brand", "color palette", "typography", "brand voice", "logo direction" |
| Map user journeys | **ux-flow** | "ux flow", "user journey", "state diagram", "user flow", "feature flow" |
| Create low-fi layouts | **wireframe** | "wireframe", "low-fi", "layout sketch", "screen layout", "ASCII mockup" |
| Design high-fi components | **ui-mockup** | "ui mockup", "high-fi", "component design", "visual design", "styled component" |
| Create icon pack | **icon-set** | "icon set", "icons", "svg icons", "icon pack", "generate icons" |
| Write UI text | **ux-copy** | "ux copy", "microcopy", "button text", "error messages", "onboarding text", "empty state" |

Once identified, follow the full workflow from that sub-skill exactly as if it had been invoked directly:

### Utility Skill Paths
- **init** → `.claude/skills/waramity/setup/init/SKILL.md`
- **sync** → `.claude/skills/waramity/setup/sync/SKILL.md`
- **update** → `.claude/skills/waramity/setup/update/SKILL.md`

### Dev Skill Paths
- **planner** → `.claude/skills/waramity/dev/planner/SKILL.md`
- **doer** → `.claude/skills/waramity/dev/doer/SKILL.md`
- **save-wip** → `.claude/skills/waramity/dev/save-wip/SKILL.md`
- **shipper** → `.claude/skills/waramity/dev/shipper/SKILL.md`
- **track** → `.claude/skills/waramity/dev/track/SKILL.md`
- **gh-remote** → `.claude/skills/waramity/dev/gh-remote/SKILL.md`
- **fail** → `.claude/skills/waramity/dev/fail/SKILL.md`

### Business Skill Paths
- **business-plan** → `.claude/skills/waramity/business/SKILL.md`
- **app-analysis** → `.claude/skills/waramity/business/app-analysis/SKILL.md`
- **keyword-trend** → `.claude/skills/waramity/business/keyword-trend/SKILL.md`
- **solo-rank** → `.claude/skills/waramity/business/solo-rank/SKILL.md`
- **work-breakdown** → `.claude/skills/waramity/business/work-breakdown/SKILL.md`

### Design Skill Paths
- **brand-kit** → `.claude/skills/waramity/design/brand-kit/SKILL.md`
- **ux-flow** → `.claude/skills/waramity/design/ux-flow/SKILL.md`
- **wireframe** → `.claude/skills/waramity/design/wireframe/SKILL.md`
- **ui-mockup** → `.claude/skills/waramity/design/ui-mockup/SKILL.md`
- **icon-set** → `.claude/skills/waramity/design/icon-set/SKILL.md`
- **ux-copy** → `.claude/skills/waramity/design/ux-copy/SKILL.md`

If the intent is ambiguous, ask the user:
> "Which workflow do you want?
> - **Utility**: init, sync (push/pull skills), or update (validate references)?
> - **Dev**: plan, do REQ, save WIP, ship, track, gh-remote, or fail REQ?
> - **Business**: business plan, app analysis, keyword trend, solo rank, or work breakdown?
> - **Design**: brand-kit, ux-flow, wireframe, ui-mockup, icon-set, or ux-copy?"
