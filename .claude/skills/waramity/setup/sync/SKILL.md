---
name: sync
description: >
  Sync waramity skills between local and remote repository (waramity/waramity-skills).
  Auto-checks status first, then suggests pull or push based on differences.
  Triggers on: "sync skills", "push skills", "pull skills", "sync waramity",
  "upload skills to github", "download latest skills".
---

# Skill: sync

Bidirectional sync between local `.claude/skills/waramity/` and remote `waramity/waramity-skills` repository.

**Auto-detects differences and suggests the right action.**

## Configuration

```
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_PATH="skills/waramity"
LOCAL_PATH=".claude/skills/waramity"
```

---

## Workflow

### Step 1: Check Status (Always First)

Compare local and remote files to determine sync state:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE=".claude/skills/waramity"

echo "Comparing local vs remote..."
echo ""

# Get remote files (blob = files only)
REMOTE_FILES=$(gh api "repos/$REPO/git/trees/$BRANCH?recursive=1" --jq '.tree[] | select(.path | startswith("skills/waramity/")) | select(.type == "blob") | .path' | sed "s|skills/waramity/||g" | sort)

# Get local files
LOCAL_FILES=$(find "$LOCAL_BASE" -type f ! -name ".*" | sed "s|$LOCAL_BASE/||g" | sort)

# Find differences
REMOTE_ONLY=$(comm -23 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES"))
LOCAL_ONLY=$(comm -13 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES"))
BOTH=$(comm -12 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES"))
```

---

### Step 2: Display Status Report

Show a summary of differences:

```
=== Sync Status ===

📥 Remote only (need to pull): N files
  - path/to/file1.md
  - path/to/file2.md

📤 Local only (need to push): N files
  - path/to/new-file.md

📁 Both exist: N files (may have content changes)
```

---

### Step 3: Suggest Action

Based on the status, suggest the appropriate action:

**Case 1: Remote has new files → Suggest Pull**
```
Remote has N new files not in local.

Suggested action: Pull
> Download new files from GitHub to local

Proceed with pull? (y/n)
```

**Case 2: Local has new files → Suggest Push**
```
Local has N new files not in remote.

Suggested action: Push
> Upload new files from local to GitHub

Proceed with push? (y/n)
```

**Case 3: Both have unique files → Ask User**
```
Both local and remote have unique files:
- Remote only: N files
- Local only: M files

What do you want to do?
1. Pull (download remote → local)
2. Push (upload local → remote)
3. Pull then Push (full sync)
```

**Case 4: All synced → Confirm**
```
✓ Local and remote are in sync!

N files exist in both locations.
No action needed.
```

---

### Step 4A: Pull (Remote → Local)

Download files from remote repo to local:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE=".claude/skills/waramity"

echo "Pulling from $REPO..."

# Get all files from remote
FILES=$(gh api "repos/$REPO/git/trees/$BRANCH?recursive=1" --jq '.tree[] | select(.path | startswith("skills/waramity/")) | select(.type == "blob") | .path')

for file in $FILES; do
  relative="${file#skills/waramity/}"
  local_path="$LOCAL_BASE/$relative"
  mkdir -p "$(dirname "$local_path")"
  gh api "repos/$REPO/contents/$file" --jq '.content' 2>/dev/null \
    | base64 -d > "$local_path" \
    && echo "✓ $relative" \
    || echo "✗ Failed: $relative"
done

echo ""
echo "Pull complete."
```

---

### Step 4B: Push (Local → Remote)

Upload local files to remote repo:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE=".claude/skills/waramity"

echo "Pushing to $REPO..."

find "$LOCAL_BASE" -type f ! -name ".*" | while read local_file; do
  relative="${local_file#$LOCAL_BASE/}"
  remote_path="$REMOTE_BASE/$relative"

  # Get current SHA if file exists
  SHA=$(gh api "repos/$REPO/contents/$remote_path" --jq '.sha' 2>/dev/null || echo "")

  # Encode content
  CONTENT=$(base64 -i "$local_file")

  # Push to GitHub
  if [ -n "$SHA" ]; then
    gh api "repos/$REPO/contents/$remote_path" --method PUT \
      -f message="Update $relative" \
      -f content="$CONTENT" \
      -f sha="$SHA" \
      -f branch="$BRANCH" >/dev/null 2>&1 \
      && echo "✓ Updated: $relative" \
      || echo "✗ Failed: $relative"
  else
    gh api "repos/$REPO/contents/$remote_path" --method PUT \
      -f message="Add $relative" \
      -f content="$CONTENT" \
      -f branch="$BRANCH" >/dev/null 2>&1 \
      && echo "✓ Added: $relative" \
      || echo "✗ Failed: $relative"
  fi
done

echo ""
echo "Push complete."
```

---

## Rules

- **Always check status first** before any sync action
- Suggest action based on differences found
- Confirm with user before push (overwrites remote)
- Pull is safe — only overwrites local files
- Use `gh` CLI for all GitHub operations
- Excludes hidden files (.*) from sync
- Show clear success/failure for each file

---

## Quick Commands

### Check status only
```
/waramity → "sync status" or "sync skills"
```

### Force pull (skip status)
```
/waramity → "pull skills" or "download skills"
```

### Force push (skip status)
```
/waramity → "push skills" or "upload skills"
```

---

## Examples

**User**: "sync skills"
→ Check status → Show differences → Suggest pull/push → Confirm → Execute

**User**: "pull skills"
→ Skip status → Pull all files from remote

**User**: "push skills"
→ Skip status → Confirm → Push all files to remote
