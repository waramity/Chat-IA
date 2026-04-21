---
name: icon-set
description: >
  Create consistent SVG icon pack.
  Reads brand-kit for icon colors if available.
  Triggers on: "icon set", "icons", "svg icons", "icon pack", "generate icons".
---

# Skill: icon-set

Create consistent SVG icons for the project.

## Workflow

### 1. Check Brand Kit

```bash
# Check if brand-kit exists
cat .waramity/design/brand-kit.md 2>/dev/null
```

If brand-kit exists, extract:
- Primary color for filled icons
- Text colors for outline icons
- Icon style preference (if defined)

### 2. Gather Context

Ask:
> "What icons do you need?"
> - **Category** - Navigation, Actions, Objects, Status, Social
> - **Specific icons** - List icon names (e.g., home, search, settings)
> - **Purpose** - Tab bar, toolbar, inline, decorative

**Style question:**
> "What icon style?"
> - **Outline** - 1.5-2px stroke, no fill (modern, clean)
> - **Filled** - Solid shapes (bold, prominent)
> - **Duotone** - Two-tone with opacity (depth, dimension)
> - **Match existing** - Reference existing icons in project

**Size question:**
> "What size?"
> - **24x24** - Standard UI icons
> - **20x20** - Compact/dense UI
> - **16x16** - Inline with text
> - **32x32** - Feature icons
> - **Multiple sizes** - Generate size variants

### 3. Generate SVG Icons

Create icons with consistent properties:

**Icon Template (Outline):**
```svg
<svg
  xmlns="http://www.w3.org/2000/svg"
  width="24"
  height="24"
  viewBox="0 0 24 24"
  fill="none"
  stroke="currentColor"
  stroke-width="2"
  stroke-linecap="round"
  stroke-linejoin="round"
>
  <!-- Icon paths here -->
</svg>
```

**Icon Template (Filled):**
```svg
<svg
  xmlns="http://www.w3.org/2000/svg"
  width="24"
  height="24"
  viewBox="0 0 24 24"
  fill="currentColor"
>
  <!-- Icon paths here -->
</svg>
```

**Example Icons:**

```svg
<!-- home.svg -->
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
  <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/>
  <polyline points="9 22 9 12 15 12 15 22"/>
</svg>

<!-- search.svg -->
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
  <circle cx="11" cy="11" r="8"/>
  <line x1="21" y1="21" x2="16.65" y2="16.65"/>
</svg>

<!-- settings.svg -->
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
  <circle cx="12" cy="12" r="3"/>
  <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"/>
</svg>
```

### 4. Quality Check

Verify consistency:
- [ ] Same viewBox (24x24)
- [ ] Same stroke-width (2px for outline)
- [ ] Same stroke-linecap/linejoin
- [ ] Visual weight balance across set
- [ ] Uses currentColor for theming

### 5. Save & Present

```bash
# Ensure directory exists
mkdir -p .waramity/design/icons

# Save each icon
# File: .waramity/design/icons/[icon-name].svg
```

Also create index file:
```bash
# File: .waramity/design/icons/index.md
```

**Index format:**
```markdown
# Icon Set

Generated: YYYY-MM-DD
Style: [Outline/Filled/Duotone]
Size: [24x24]
Count: [N] icons

## Icons

| Icon | Name | File |
|------|------|------|
| 🏠 | home | home.svg |
| 🔍 | search | search.svg |
| ⚙️ | settings | settings.svg |

## Usage

### React/React Native
\`\`\`jsx
import { Home } from './icons';
<Home width={24} height={24} color="#000" />
\`\`\`

### SwiftUI
\`\`\`swift
Image("home")
  .resizable()
  .frame(width: 24, height: 24)
  .foregroundColor(.primary)
\`\`\`

### CSS
\`\`\`css
.icon {
  width: 24px;
  height: 24px;
  color: currentColor;
}
\`\`\`
```

---

## Common Icon Categories

| Category | Icons |
|----------|-------|
| Navigation | home, back, forward, menu, close, chevron |
| Actions | add, edit, delete, share, download, upload |
| Objects | file, folder, image, video, document, link |
| Status | check, warning, error, info, loading |
| Communication | mail, chat, bell, phone, send |
| User | user, users, settings, logout, profile |
| Media | play, pause, stop, volume, mic |
| Commerce | cart, bag, credit-card, gift, tag |

---

## Rules

- Use currentColor for easy theming
- Maintain consistent stroke width
- Icons should work at target size (no tiny details)
- Check brand-kit for color guidance
- Generate React component wrapper if requested
- SVG should be optimized (no unnecessary attributes)
