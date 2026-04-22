---
name: install-skill
description: >
  Browse and install skills from the waramity skill registry.
  Downloads skills from GitHub repos and auto-connects to waramity categories.
  Tracks installed skills in .waramity/memory/installed-skill/.
  Triggers on: "install skill", "list skills", "available skills", "what skills can I install",
  "show skill registry", "add skill", "install ui-ux-pro-max".
---

# Skill: install-skill

Browse available skills from the registry, install from GitHub, and auto-connect to waramity skills.

## Skill Registry

The skill registry is stored in a separate file for easy maintenance:

```
.claude/skills/waramity/setup/install-skill/registry.md
```

**To read the registry:**
```bash
cat .claude/skills/waramity/setup/install-skill/registry.md
```

> **Note**: To add/remove skills, edit `registry.md` in this folder.

---

## Workflow

### Step 1: Determine Action

Parse user request to determine action:
- **list** — Show available skills from registry
- **install** — Install a specific skill
- **installed** — Show already installed skills
- **uninstall** — Remove an installed skill

If unclear, ask:
> "What would you like to do?
> 1. **List** — Show available skills to install
> 2. **Install** — Install a skill (by name or number)
> 3. **Installed** — Show what's already installed
> 4. **Uninstall** — Remove an installed skill"

---

### Step 2A: List Available Skills

Show the registry table above with installation status:

```bash
# Check which skills are already installed
INSTALLED_DIR=".waramity/memory/installed-skill"

echo "=== Available Skills ==="
echo ""
echo "| # | Name | Description | Status |"
echo "|---|------|-------------|--------|"

# For each skill in registry, check if installed
# Example output:
# | 1 | ui-ux-pro-max | UI/UX design intelligence... | Installed |
# | 2 | swift-ios-skills | Swift/iOS development... | Not installed |
```

Read the registry from this SKILL.md and cross-reference with `.waramity/memory/installed-skill/` to show status.

---

### Step 2B: Install a Skill

When user specifies a skill to install (by name or number):

#### Step 2B.1: Validate Selection

Match user input to registry:
- "1" or "ui-ux-pro-max" → skill ID 1
- "install swift" → partial match to "swift-ios-skills"

If no match, show the list and ask for clarification.

#### Step 2B.2: Clone from GitHub

```bash
SKILL_NAME="ui-ux-pro-max"
REPO="waramity/ui-ux-pro-max"
INSTALL_PATH=".claude/skills/external/$SKILL_NAME"

# Check if already exists
if [ -d "$INSTALL_PATH" ]; then
  echo "Skill already exists at $INSTALL_PATH"
  echo "Use 'uninstall $SKILL_NAME' first to reinstall"
  exit 0
fi

# Clone the repo
mkdir -p ".claude/skills/external"
gh repo clone "$REPO" "$INSTALL_PATH" -- --depth 1

# Verify SKILL.md exists
if [ ! -f "$INSTALL_PATH/SKILL.md" ]; then
  echo "Warning: No SKILL.md found in repo"
fi

echo "Downloaded: $SKILL_NAME"
```

#### Step 2B.3: Record Installation

Create installation record in `.waramity/memory/installed-skill/`:

```bash
mkdir -p ".waramity/memory/installed-skill"
```

Create file: `.waramity/memory/installed-skill/{skill-name}.md`

**File template:**

```markdown
# Installed Skill: {skill-name}

## Installation Info

**Name**: {skill-name}
**Repo**: {github-repo}
**Installed**: {date}
**Path**: .claude/skills/external/{skill-name}
**Connects To**: {waramity-category}

## Source

- **GitHub**: https://github.com/{repo}
- **Branch**: main

## Status

- [x] Downloaded
- [ ] Connected to waramity

---

*Installed on: {date}*
```

#### Step 2B.4: Auto-Connect to Waramity

After installation, automatically trigger the `connect` skill:

> "Now connecting **{skill-name}** to **{waramity-category}** skill..."

Execute the connect workflow:
1. Read `.claude/skills/external/{skill-name}/SKILL.md`
2. Read `.claude/skills/waramity/{category}/SKILL.md`
3. Create connection file at `.waramity/memory/connected-skill/{category}--{skill-name}.md`

Update installation record:
```markdown
## Status

- [x] Downloaded
- [x] Connected to waramity
```

#### Step 2B.5: Confirm Installation

Output summary:

```
Installation complete!

Skill: {skill-name}
Path: .claude/skills/external/{skill-name}
Connected to: {waramity-category}

Files created:
- .waramity/memory/installed-skill/{skill-name}.md
- .waramity/memory/connected-skill/{category}--{skill-name}.md

To use: /waramity → "{trigger phrase}"
```

---

### Step 2C: Show Installed Skills

List all installed skills:

```bash
INSTALLED_DIR=".waramity/memory/installed-skill"

if [ ! -d "$INSTALLED_DIR" ] || [ -z "$(ls -A $INSTALLED_DIR 2>/dev/null)" ]; then
  echo "No skills installed yet."
  echo "Run 'install skill list' to see available skills."
  exit 0
fi

echo "=== Installed Skills ==="
echo ""

for file in "$INSTALLED_DIR"/*.md; do
  if [ -f "$file" ]; then
    SKILL_NAME=$(basename "$file" .md)
    INSTALLED_DATE=$(grep "^\*Installed on:" "$file" | sed 's/\*Installed on: //' | sed 's/\*//')
    CONNECTS_TO=$(grep "^**Connects To**:" "$file" | sed 's/**Connects To**: //')
    echo "- $SKILL_NAME (→ $CONNECTS_TO) — installed $INSTALLED_DATE"
  fi
done
```

---

### Step 2D: Uninstall a Skill

Remove an installed skill:

```bash
SKILL_NAME="ui-ux-pro-max"
INSTALL_PATH=".claude/skills/external/$SKILL_NAME"
RECORD_FILE=".waramity/memory/installed-skill/$SKILL_NAME.md"

# Get connected category from record
CATEGORY=$(grep "^**Connects To**:" "$RECORD_FILE" | sed 's/**Connects To**: //')
CONNECTION_FILE=".waramity/memory/connected-skill/$CATEGORY--$SKILL_NAME.md"

# Confirm with user
echo "This will remove:"
echo "  - $INSTALL_PATH (skill files)"
echo "  - $RECORD_FILE (installation record)"
echo "  - $CONNECTION_FILE (connection mapping)"
echo ""
echo "Proceed? (y/n)"
```

After confirmation:

```bash
rm -rf "$INSTALL_PATH"
rm -f "$RECORD_FILE"
rm -f "$CONNECTION_FILE"
echo "Uninstalled: $SKILL_NAME"
```

---

## Rules

- Always check if skill is already installed before downloading
- Use `gh repo clone` for GitHub operations (requires gh CLI)
- Clone with `--depth 1` to minimize download size
- Auto-connect after installation (don't wait for user)
- Store skills in `.claude/skills/external/` (not in waramity folder)
- Record all installations in `.waramity/memory/installed-skill/`
- Confirm before uninstalling (destructive operation)

---

## Quick Commands

### List available skills
```
/waramity → "list skills" or "available skills"
```

### Install a skill
```
/waramity → "install ui-ux-pro-max"
/waramity → "install skill 1"
```

### Show installed skills
```
/waramity → "show installed skills" or "what's installed"
```

### Uninstall a skill
```
/waramity → "uninstall ui-ux-pro-max"
```

---

## Examples

**User**: "what skills can I install"
→ Show registry table with status column

**User**: "install ui-ux-pro-max"
→ Clone repo → Record installation → Auto-connect to design → Confirm

**User**: "install 2"
→ Look up ID 2 in registry → Install swift-ios-skills → Connect to dev

**User**: "uninstall swift-ios-skills"
→ Confirm → Remove files and records
