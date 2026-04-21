---
name: wireframe
description: >
  Create low-fidelity layouts using ASCII art.
  Can reference ux-flow diagrams.
  Triggers on: "wireframe", "low-fi", "layout sketch", "screen layout", "ASCII mockup".
---

# Skill: wireframe

Create low-fidelity screen layouts using ASCII art.

## Workflow

### 1. Gather Context

Check for existing ux-flows:
```bash
ls .waramity/design/ux-flows/*.md 2>/dev/null
```

Ask:
> "What screen do you want to wireframe?"
> - Provide screen name/purpose
> - Reference existing ux-flow (if any)
> - Describe key elements needed

**Platform question:**
> "What platform?"
> - **Mobile** (375x812 - iPhone proportions)
> - **Tablet** (768x1024 - iPad proportions)
> - **Desktop** (1440x900 - Standard desktop)
> - **Responsive** (Show mobile + desktop)

### 2. Identify Layout Elements

Based on user input, identify:
- **Header/Nav** - Navigation, title, actions
- **Content areas** - Main content blocks
- **Interactive elements** - Buttons, inputs, lists
- **Footer** - Bottom navigation, actions

### 3. Generate ASCII Wireframe

Use box-drawing characters for clean wireframes:

**Mobile Screen Example:**
```
┌─────────────────────────────┐
│  ← Back     Title     [···] │  <- Header
├─────────────────────────────┤
│                             │
│  ┌───────────────────────┐  │
│  │                       │  │
│  │      Hero Image       │  │
│  │                       │  │
│  └───────────────────────┘  │
│                             │
│  Heading Text               │
│  ─────────────              │
│  Body text goes here with   │
│  description of the feature │
│                             │
│  ┌───────────────────────┐  │
│  │    Primary Button     │  │
│  └───────────────────────┘  │
│                             │
│  [ ] Checkbox option        │
│  [ ] Another option         │
│                             │
├─────────────────────────────┤
│  [🏠]   [🔍]   [👤]   [⚙]  │  <- Tab Bar
└─────────────────────────────┘
```

**Form Elements:**
```
┌─ Label ─────────────────────┐
│  Placeholder text           │
└─────────────────────────────┘

┌─ Dropdown ──────────────────┐
│  Selected value           ▼ │
└─────────────────────────────┘

[○] Radio option 1
[●] Radio option 2 (selected)

[✓] Checkbox (checked)
[ ] Checkbox (unchecked)
```

**List Items:**
```
┌─────────────────────────────┐
│ [IMG]  Title                │
│        Subtitle         [→] │
├─────────────────────────────┤
│ [IMG]  Title                │
│        Subtitle         [→] │
└─────────────────────────────┘
```

**Cards:**
```
┌─────────────────────────────┐
│ ┌─────────────────────────┐ │
│ │       Card Image        │ │
│ └─────────────────────────┘ │
│                             │
│  Card Title                 │
│  Card description text      │
│                             │
│  [Action]        [Action 2] │
└─────────────────────────────┘
```

### 4. Add Annotations

Include with wireframe:
```markdown
## Annotations

1. **Header** - Fixed, back nav + actions
2. **Hero** - 16:9 aspect ratio image
3. **CTA Button** - Primary action, full width
4. **Tab Bar** - 4 items, home selected
```

### 5. Save & Present

```bash
# Ensure directory exists
mkdir -p .waramity/design/wireframes

# Save wireframe
# File: .waramity/design/wireframes/[screen-name].md
```

**Output format:**
```markdown
# Wireframe: [Screen Name]

Generated: YYYY-MM-DD
Platform: [Mobile/Tablet/Desktop]
Related Flow: [Link to ux-flow if any]

## Layout

\`\`\`
[ASCII wireframe here]
\`\`\`

## Annotations

1. **[Element]** - [Description, behavior]
2. **[Element]** - [Description, behavior]
...

## Interactions

| Element | Tap/Click | Swipe | Long Press |
|---------|-----------|-------|------------|
| [Name] | [Action] | - | - |

## Notes

- [Design decisions]
- [Constraints]
```

---

## ASCII Components Library

### Navigation
```
← Back     Title     [···]     // Header with actions
[🏠] [🔍] [➕] [👤] [⚙]        // Tab bar
≡ Menu                         // Hamburger
```

### Buttons
```
┌───────────────────────┐
│    Primary Button     │      // Filled button
└───────────────────────┘

┌───────────────────────┐
│    Secondary Btn      │      // Outlined button
└───────────────────────┘

[ Link Button ]                // Text button
```

### Status
```
[====----] 60%                 // Progress bar
[●●●○○]                        // Step indicator
(!) Warning                    // Alert
```

---

## Rules

- Use consistent box-drawing characters
- Include annotations for every major element
- Reference ux-flow when available
- Keep proportions realistic for platform
- Show one state per wireframe (create variants for different states)
- This is low-fi - no colors, no exact spacing
