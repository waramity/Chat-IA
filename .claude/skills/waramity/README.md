# waramity-skills

Custom skill system for requirement-driven development and business planning. Organized into three categories: **business**, **dev**, and **setup**.

---

## Full Pipeline

One feature end-to-end workflow:

```
app-analysis → business-plan → keyword-trend → solo-rank → brand-kit¹
     ↓
ux-flow → wireframe → ui-mockup → ux-copy → work-breakdown
     ↓
planner → doer → test-plan → deploy-check → shipper
     ↓
landing-page + app-store-listing → social-post + blog-post
     ↓
faq-builder → feedback-triage → loops back to planner
```

**Stages:**
1. **Discovery**: `app-analysis` → `business-plan` → `keyword-trend` → `solo-rank`
2. **Branding**: `brand-kit`¹
3. **Design**: `ux-flow` → `wireframe` → `ui-mockup` → `ux-copy`
4. **Planning**: `work-breakdown` → `planner`
5. **Development**: `doer` → `test-plan` → `deploy-check` → `shipper`
6. **Launch**: `landing-page` + `app-store-listing`
7. **Marketing**: `social-post` + `blog-post`
8. **Support**: `faq-builder` → `feedback-triage`
9. **Iterate**: loops back to `planner`

¹ *brand-kit and other unlisted skills are planned/coming soon*

---

## Project Structure

```
.waramity/
└── dev/
    ├── requirement/     # REQ-NNNN.md files (planned tasks)
    └── wip/             # Work-in-progress commits

skills/
└── waramity/
    ├── SKILL.md         # Main router
    ├── business/        # Business planning & analysis skills
    │   ├── SKILL.md     # Business router
    │   ├── app-analysis/
    │   ├── keyword-trend/
    │   ├── solo-rank/
    │   └── work-breakdown/
    ├── dev/             # Development workflow skills
    │   ├── SKILL.md     # Dev router
    │   ├── doer/
    │   ├── fail/
    │   ├── gh-remote/
    │   ├── planner/
    │   ├── save-wip/
    │   ├── shipper/
    │   └── track/
    └── setup/           # Setup & maintenance skills
        ├── SKILL.md     # Setup router
        ├── init/
        ├── sync/
        └── update/
```

---

## Categories

### Business Skills

Analysis and planning tools for app ideas and business opportunities.

| Sub-skill | Trigger Phrases | Description |
|-----------|-----------------|-------------|
| **app-analysis** | "analyze app idea", "app business plan", "tech stack for app" | Deep business analysis for a single app idea with interactive widget |
| **keyword-trend** | "keyword trend app", "rank app ideas", "Google Trends style" | Google Trends-style ranking widget for app ideas |
| **solo-rank** | "low risk app", "solo developer app", "passive income app" | Solo-developer friendliness ranking with 5-axis scoring |
| **work-breakdown** | "work breakdown", "WBS", "break this down", "split tasks" | Generate WBS with actionable Claude Code prompts |

### Dev Skills

Requirement-driven development workflow.

| Sub-skill | Trigger Phrases | Description |
|-----------|-----------------|-------------|
| **planner** | "plan this", "analyze this", "let's plan" | Analyze task and create structured REQ file |
| **doer** | "do REQ-NNNN", "implement REQ", "work on REQ" | Implement a planned requirement step by step |
| **save-wip** | "checkpoint this", "save WIP", "save progress" | Commit unfinished work with full context |
| **shipper** | "ship REQ-NNNN", "commit and push", "finish REQ" | Commit and push completed requirement cleanly |
| **track** | "track changes", "what should I commit" | Analyze uncommitted files and suggest groupings |
| **fail** | "fail REQ", "abandon REQ", "revert REQ" | Revert code changes and delete REQ file |
| **gh-remote** | "gh-remote", "do on remote", "via gh" | GitHub operations via gh CLI without local git |

### Setup Skills

Installation and maintenance utilities.

| Sub-skill | Trigger Phrases | Description |
|-----------|-----------------|-------------|
| **init** | "init waramity", "setup waramity", "initialize" | Initialize `.waramity/dev/` folder and sync latest skills |
| **sync** | "sync skills", "pull skills", "push skills" | Bidirectional sync between local and remote repository |
| **update** | "check skills", "validate skills", "fix skill paths" | Validate and fix skill router references |

---

## Usage in Claude Code

```
# Business
"analyze this app idea"
"rank app ideas Google Trends style"
"find low-risk apps for solo developer"
"break down this project into tasks"

# Dev workflow
"plan this feature"
"do REQ-0003"
"checkpoint this — it's half done"
"ship REQ-0003"
"track changes"
"fail REQ-0003"
"gh-remote view PR 42"

# Setup
"init waramity"
"sync skills"
"check skills"
```

---

## Installation

**Project-local:**

```bash
git clone git@github.com:waramity/waramity-skills.git /tmp/waramity-skills
cp -r /tmp/waramity-skills/skills/ your-project/.claude/skills/
cp -r /tmp/waramity-skills/.waramity/ your-project/.waramity/
```

**Global (all projects):**

```bash
cp -r /tmp/waramity-skills/skills/ ~/.claude/skills/
```

---

## REQ Workflow States

```
Planned  →  WIP  →  Wait to Review  →  Done
              ↓
           Failed (via fail skill)
```

---

## File Conventions

| File | Location | Purpose |
|------|----------|---------|
| `REQ-NNNN.md` | `.waramity/dev/requirement/` | Planned requirements with specs |
| `WIP-NNNN.md` | `.waramity/dev/wip/` | Work-in-progress context snapshots |
| `SKILL.md` | Each skill folder | Skill definition and workflow |
