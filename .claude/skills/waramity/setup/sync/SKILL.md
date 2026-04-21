---
name: sync
description: >
  Sync waramity skills between local and remote repository (waramity/waramity-skills).
  Triggers on: "sync skills", "push skills", "pull skills", "sync waramity",
  "upload skills to github", "download latest skills", "push local to remote",
  "pull remote to local", "pull new skill", "push skill to github".
---

# Skill: sync

Bidirectional sync between local `.claude/skills/waramity/` and remote `waramity/waramity-skills` repository.

Syncs **all files** in the waramity skill folder.

## Configuration

```
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_PATH="skills/waramity"
LOCAL_PATH="$HOME/.claude/skills/waramity"  # or project-local path
```

---

## Workflow

### Step 1: Detect Direction

Ask the user:
> "What do you want to do?
> 1. **Pull** — Download latest skills from GitHub to local
> 2. **Push** — Upload local skills to GitHub
> 3. **Status** — Compare local vs remote (no changes)"

---

### Step 2A: Pull (Remote → Local)

Download all files from remote repo to local:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE="$HOME/.claude/skills/waramity"

# Function to download a file
download_file() {
  local remote_path="$1"
  local local_path="$2"
  mkdir -p "$(dirname "$local_path")"
  gh api "repos/$REPO/contents/$remote_path" --jq '.content' 2>/dev/null \
    | base64 -d > "$local_path" \
    && echo "✓ Downloaded: $local_path" \
    || echo "✗ Failed: $remote_path"
}

# Get all files from remote (type=blob means files only, not directories)
echo "Fetching file list from $REPO..."
FILES=$(gh api "repos/$REPO/git/trees/$BRANCH?recursive=1" --jq '.tree[] | select(.path | startswith("skills/waramity/")) | select(.type == "blob") | .path')

for file in $FILES; do
  relative="${file#skills/waramity/}"
  download_file "$file" "$LOCAL_BASE/$relative"
done

echo ""
echo "Pull complete."
```

---

### Step 2B: Push (Local → Remote)

Upload all local files to remote repo:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE="$HOME/.claude/skills/waramity"

# Function to upload a file
upload_file() {
  local local_path="$1"
  local remote_path="$2"

  # Get current SHA if file exists
  SHA=$(gh api "repos/$REPO/contents/$remote_path" --jq '.sha' 2>/dev/null || echo "")

  # Encode content
  CONTENT=$(base64 -i "$local_path")

  # Build JSON payload
  if [ -n "$SHA" ]; then
    JSON=$(jq -n --arg msg "Update $remote_path" --arg content "$CONTENT" --arg sha "$SHA" --arg branch "$BRANCH" \
      '{message: $msg, content: $content, sha: $sha, branch: $branch}')
  else
    JSON=$(jq -n --arg msg "Add $remote_path" --arg content "$CONTENT" --arg branch "$BRANCH" \
      '{message: $msg, content: $content, branch: $branch}')
  fi

  # Push to GitHub
  echo "$JSON" | gh api "repos/$REPO/contents/$remote_path" --method PUT --input - > /dev/null 2>&1 \
    && echo "✓ Pushed: $remote_path" \
    || echo "✗ Failed: $remote_path"
}

# Find all local files (excluding .git and hidden files)
echo "Scanning local skills..."
find "$LOCAL_BASE" -type f ! -name ".*" | while read local_file; do
  relative="${local_file#$LOCAL_BASE/}"
  remote_path="$REMOTE_BASE/$relative"
  upload_file "$local_file" "$remote_path"
done

echo ""
echo "Push complete."
```

---

### Step 2C: Status (Compare)

Show differences between local and remote:

```bash
REPO="waramity/waramity-skills"
BRANCH="master"
REMOTE_BASE="skills/waramity"
LOCAL_BASE="$HOME/.claude/skills/waramity"

echo "Comparing local vs remote..."
echo ""

# Get remote files (blob = files only)
REMOTE_FILES=$(gh api "repos/$REPO/git/trees/$BRANCH?recursive=1" --jq '.tree[] | select(.path | startswith("skills/waramity/")) | select(.type == "blob") | .path' | sed "s|skills/waramity/||g" | sort)

# Get local files
LOCAL_FILES=$(find "$LOCAL_BASE" -type f ! -name ".*" | sed "s|$LOCAL_BASE/||g" | sort)

echo "=== Remote only (need to pull) ==="
comm -23 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES")

echo ""
echo "=== Local only (need to push) ==="
comm -13 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES")

echo ""
echo "=== Both exist (may need sync) ==="
comm -12 <(echo "$REMOTE_FILES") <(echo "$LOCAL_FILES")
```

---

## Quick Commands

### Pull all files
```bash
gh api "repos/waramity/waramity-skills/git/trees/master?recursive=1" --jq '.tree[] | select(.path | startswith("skills/waramity/")) | select(.type == "blob") | .path' | while read file; do
  rel="${file#skills/waramity/}"
  mkdir -p "$HOME/.claude/skills/waramity/$(dirname "$rel")"
  gh api "repos/waramity/waramity-skills/contents/$file" --jq '.content' | base64 -d > "$HOME/.claude/skills/waramity/$rel"
  echo "✓ $rel"
done
```

### Push single file
```bash
FILE_PATH="business/business-plan/template.md"
CONTENT=$(base64 -i "$HOME/.claude/skills/waramity/$FILE_PATH")
SHA=$(gh api "repos/waramity/waramity-skills/contents/skills/waramity/$FILE_PATH" --jq '.sha' 2>/dev/null || echo "")
if [ -n "$SHA" ]; then
  gh api "repos/waramity/waramity-skills/contents/skills/waramity/$FILE_PATH" --method PUT \
    -f message="Update $FILE_PATH" -f content="$CONTENT" -f sha="$SHA" -f branch="master"
else
  gh api "repos/waramity/waramity-skills/contents/skills/waramity/$FILE_PATH" --method PUT \
    -f message="Add $FILE_PATH" -f content="$CONTENT" -f branch="master"
fi
```

---

## Rules

- Always confirm with user before push (uploading overwrites remote)
- Pull is safe — only overwrites local files
- Use `gh` CLI for all GitHub operations (per CLAUDE.md)
- Syncs all files (not just SKILL.md)
- Excludes hidden files (.*) from sync
- Show clear success/failure for each file

---

## Examples

**User**: "sync skills pull"
→ Pull all files from remote to local

**User**: "push skills to github"
→ Push all local files to remote

**User**: "sync status"
→ Show comparison without making changes

**User**: "push only business/business-plan/template.md"
→ Push single file
