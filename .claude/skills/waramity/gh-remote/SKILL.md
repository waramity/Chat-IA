---
name: gh-remote
description: Perform all GitHub operations exclusively through the gh CLI — no local git commands, no checkout, no staging. Handles PRs, issues, branch management, and releases entirely against the remote. Trigger on phrases like "do on remote", "gh-remote", "operate remotely", "no local", "via gh", or when acting on an existing remote branch or PR without touching the local working tree.
---

# Skill: gh-remote

Perform GitHub operations purely via `gh` CLI against the remote. No local git commands, no checkout, no staging.

## Trigger Phrases

Trigger this skill when the user says:
- "do on remote"
- "gh remote"
- "gh-remote"
- "operate remotely"
- "no local"
- "via gh"
- Acting on PR/branch number with "remote-only" intent

## Quick Start

Always resolve owner/repo dynamically:
```bash
rtk gh repo view --json nameWithOwner -q .nameWithOwner
```

Use this in subsequent `gh api` calls as `{owner}/{repo}`.

---

## PR Operations

All PR operations use `gh pr` commands. Accept PR number or branch name as identifier.

### Create PR
```bash
rtk gh pr create --title "Title" --body "Body" --base main --head feature-branch
```

### View PR
```bash
rtk gh pr view 123
rtk gh pr view feature-branch
```

### List PRs
```bash
rtk gh pr list
rtk gh pr list --state open
rtk gh pr list --state closed --limit 10
```

### Review PR
```bash
rtk gh pr review 123 --approve
rtk gh pr review 123 --request-changes --body "needs work"
```

### Merge PR
```bash
rtk gh pr merge 123 --squash
rtk gh pr merge 123 --rebase
rtk gh pr merge 123 --merge
```

### Close PR
```bash
rtk gh pr close 123
```

### Reopen PR
```bash
rtk gh pr reopen 123
```

---

## Issue Operations

All issue operations use `gh issue` commands.

### Create Issue
```bash
rtk gh issue create --title "Bug: something broken" --body "Description here"
```

### View Issue
```bash
rtk gh issue view 456
```

### List Issues
```bash
rtk gh issue list
rtk gh issue list --state open --label bug
```

### Close Issue
```bash
rtk gh issue close 456
```

### Reopen Issue
```bash
rtk gh issue reopen 456
```

### Add Label to Issue
```bash
rtk gh issue edit 456 --add-label bug
rtk gh issue edit 456 --add-label "help wanted"
```

### Link Issue to Branch (develop)
```bash
rtk gh issue develop 456 --name feature-branch --base main
```

---

## Branch Operations

Branch operations use `gh api` REST endpoints. No local `git` commands.

### List Remote Branches
```bash
rtk gh api repos/{owner}/{repo}/branches --paginate
```

### View Branch Details
```bash
rtk gh api repos/{owner}/{repo}/git/refs/heads/{branch}
```

### Delete Remote Branch
```bash
rtk gh api -X DELETE repos/{owner}/{repo}/git/refs/heads/{branch}
```

**Note:** Replace `{owner}/{repo}` with actual repo from `gh repo view --json nameWithOwner -q .nameWithOwner`. Replace `{branch}` with the branch name.

---

## Release Operations

All release operations use `gh release` commands.

### Create Release
**Prerequisite:** Tag must exist first!

```bash
# First create the tag (remotely via gh api)
rtk gh api repos/{owner}/{repo}/git/refs -f ref="refs/tags/v1.0.0" -f sha="{commit_sha}"

# Then create the release
rtk gh release create v1.0.0 --title "Release 1.0.0" --notes "Release notes here"
```

### View Release
```bash
rtk gh release view v1.0.0
```

### List Releases
```bash
rtk gh release list
```

### Delete Release
```bash
rtk gh release delete v1.0.0 --yes
```

---

## Rules

### Hard Constraints
1. **NEVER run `git` commands** — only `gh` CLI and `gh api`
2. **NEVER `cd` or checkout** — operate purely on remote
3. **Always use `rtk` prefix** per CLAUDE.md golden rule
4. **Always resolve owner/repo** from `gh repo view --json nameWithOwner`
5. **Accept branch name or PR number** as primary identifier

### What This Skill Does NOT Do
- ❌ Create local branches
- ❌ Make local commits
- ❌ Stage files
- ❌ Switch branches
- ❌ Push to remotes (use `gh pr create` instead)

### When to Redirect
If the user needs local work (editing files, committing locally), redirect them to:
- **planner** — to plan the work
- **doer** — to implement the planned requirement

---

## Examples

### Review PR #42 without checkout
```bash
rtk gh pr view 42
rtk gh pr review 42 --approve
```

### Close issue #456 and label it
```bash
rtk gh issue edit 456 --add-label wontfix
rtk gh issue close 456
```

### Delete stale remote branch
```bash
REPO=$(rtk gh repo view --json nameWithOwner -q .nameWithOwner)
rtk gh api -X DELETE repos/$REPO/git/refs/heads/stale-branch
```

### Create release for existing tag
```bash
rtk gh release create v1.2.3 --title "Version 1.2.3" --notes "Bug fixes and improvements"
```
