---
name: ui-mockup
description: >
  Create high-fidelity mockups as React/JSX components.
  Reads brand-kit for colors, typography, spacing.
  Triggers on: "ui mockup", "high-fi", "component design", "visual design", "styled component".
---

# Skill: ui-mockup

Create high-fidelity UI components as React/JSX (SwiftUI-convertible).

## Workflow

### 1. Check Brand Kit

```bash
# Check if brand-kit exists
cat .waramity/design/brand-kit.md 2>/dev/null
```

**If no brand-kit:**
> "No brand-kit found. Do you want to:
> 1. **Create brand-kit first** (recommended for consistency)
> 2. **Proceed with defaults** (neutral colors, system fonts)"

**If brand-kit exists:**
Extract and use:
- Color palette
- Typography scale
- Spacing system
- Border radius tokens

### 2. Gather Context

Ask:
> "What component do you want to design?"
> - Button, Input, Card, Modal, List Item, etc.
> - Or describe custom component

**State variants:**
> "Which states should I include?"
> - Default, Hover, Active, Disabled
> - Loading, Success, Error
> - Empty, Filled

### 3. Generate React/JSX Component

Create component using brand-kit tokens:

**Example Button Component:**
```jsx
// Button.jsx
// Generated from brand-kit: [Project Name]

const Button = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  children,
  onPress
}) => {
  const styles = {
    primary: {
      backgroundColor: '#6366F1', // brand.primary
      color: '#FFFFFF',
    },
    secondary: {
      backgroundColor: 'transparent',
      borderWidth: 1,
      borderColor: '#6366F1',
      color: '#6366F1',
    },
  };

  const sizes = {
    sm: { paddingVertical: 8, paddingHorizontal: 16, fontSize: 14 },
    md: { paddingVertical: 12, paddingHorizontal: 24, fontSize: 16 },
    lg: { paddingVertical: 16, paddingHorizontal: 32, fontSize: 18 },
  };

  return (
    <Pressable
      style={[
        styles[variant],
        sizes[size],
        { borderRadius: 8, alignItems: 'center' },
        disabled && { opacity: 0.5 }
      ]}
      disabled={disabled || loading}
      onPress={onPress}
    >
      {loading ? (
        <ActivityIndicator color={styles[variant].color} />
      ) : (
        <Text style={{
          color: styles[variant].color,
          fontSize: sizes[size].fontSize,
          fontWeight: '600'
        }}>
          {children}
        </Text>
      )}
    </Pressable>
  );
};
```

**SwiftUI Conversion Notes:**
```swift
// SwiftUI equivalent structure:
// - Pressable → Button
// - style props → .modifiers
// - children → label closure

Button(action: onPress) {
  Text(children)
    .font(.system(size: 16, weight: .semibold))
    .foregroundColor(.white)
}
.padding(.horizontal, 24)
.padding(.vertical, 12)
.background(Color("Primary"))
.cornerRadius(8)
.disabled(disabled)
```

### 4. Include Design Tokens

Add token reference at top of file:
```jsx
// Design Tokens (from brand-kit)
const tokens = {
  colors: {
    primary: '#6366F1',
    secondary: '#8B5CF6',
    background: '#FFFFFF',
    surface: '#F9FAFB',
    textPrimary: '#111827',
    textSecondary: '#6B7280',
    border: '#E5E7EB',
    success: '#10B981',
    error: '#EF4444',
  },
  spacing: {
    xs: 4, sm: 8, md: 16, lg: 24, xl: 32,
  },
  radius: {
    sm: 4, md: 8, lg: 16, full: 9999,
  },
  typography: {
    h1: { size: 32, weight: '700', lineHeight: 1.2 },
    h2: { size: 24, weight: '600', lineHeight: 1.3 },
    body: { size: 16, weight: '400', lineHeight: 1.5 },
    small: { size: 14, weight: '400', lineHeight: 1.4 },
  },
};
```

### 5. Save & Present

```bash
# Ensure directory exists
mkdir -p .waramity/design/mockups

# Save component
# File: .waramity/design/mockups/[ComponentName].jsx
```

**Output format:**
```markdown
# UI Mockup: [Component Name]

Generated: YYYY-MM-DD
Brand Kit: [Link to brand-kit.md]

## Preview

[Description of what it looks like]

## Component Code

\`\`\`jsx
[Full React/JSX code]
\`\`\`

## Variants

| Variant | Description |
|---------|-------------|
| primary | Main action button |
| secondary | Secondary action |
| disabled | Inactive state |

## SwiftUI Conversion

\`\`\`swift
[SwiftUI equivalent]
\`\`\`

## Usage

\`\`\`jsx
<Button variant="primary" size="md" onPress={handlePress}>
  Get Started
</Button>
\`\`\`
```

---

## Component Categories

| Category | Components |
|----------|------------|
| Actions | Button, IconButton, FAB, Link |
| Inputs | TextField, TextArea, Checkbox, Radio, Switch, Slider |
| Display | Card, List, Avatar, Badge, Chip, Tag |
| Feedback | Alert, Toast, Modal, Dialog, Tooltip |
| Navigation | TabBar, NavBar, Drawer, Breadcrumb |
| Layout | Container, Grid, Stack, Divider |

---

## Rules

- Always check brand-kit first
- Use design tokens, not hardcoded values
- Include all requested state variants
- Provide SwiftUI conversion notes
- Components should be functional, not just visual
- Match React Native patterns for easier SwiftUI conversion
