---
name: design
description: >
  Design workflow skills for full-stack design system.
  Routes to: brand-kit, ux-flow, wireframe, ui-mockup, icon-set, ux-copy.
  Triggers on: "brand kit", "ux flow", "wireframe", "ui mockup", "icon set",
  "ux copy", or design-related requests.
---

# Skill: design

Router for waramity design workflow operations.

## Sub-Skills

| Sub-Skill | Purpose | Output | Trigger Phrases |
|-----------|---------|--------|-----------------|
| **brand-kit** | Define colors, typography, voice, logo direction | `.waramity/design/brand-kit.md` | "brand kit", "define brand", "color palette", "typography", "brand voice" |
| **ux-flow** | User journeys and state diagrams | Mermaid diagrams in `.md` | "ux flow", "user journey", "state diagram", "user flow" |
| **wireframe** | Low-fi layouts | ASCII art wireframes | "wireframe", "low-fi", "layout sketch", "screen layout" |
| **ui-mockup** | High-fi mockups referencing brand-kit | React/JSX components | "ui mockup", "high-fi", "component design", "visual design" |
| **icon-set** | Consistent SVG icon pack | SVG files | "icon set", "icons", "svg icons", "icon pack" |
| **ux-copy** | Microcopy for UI elements | `.md` with copy guidelines | "ux copy", "microcopy", "button text", "error messages" |

---

## Routing Logic

### Route to `brand-kit` when:
- User wants to define or update brand identity
- User mentions colors, typography, brand voice, or logo direction
- Starting a new project's design system
- No brand-kit exists and user requests design work

### Route to `ux-flow` when:
- User wants to map user journeys or feature flows
- User mentions state diagrams, flow charts, user paths
- Planning user interactions before wireframing

### Route to `wireframe` when:
- User wants quick layout sketches
- User says "wireframe", "low-fi", "rough layout"
- Text-based layout exploration needed

### Route to `ui-mockup` when:
- User wants high-fidelity component design
- User says "mockup", "high-fi", "styled component"
- Brand-kit exists and user wants visual implementation

### Route to `icon-set` when:
- User wants to create or extend icon library
- User mentions "icons", "svg", "icon pack"
- Creating consistent iconography for the project

### Route to `ux-copy` when:
- User needs UI text, microcopy, or content guidelines
- User mentions button text, error messages, empty states
- Writing user-facing copy that matches brand voice

---

## Skill Dependencies

```
brand-kit (foundation)
    |
    +-- ui-mockup (reads brand-kit)
    +-- icon-set (reads brand-kit for colors)
    +-- ux-copy (reads brand-kit for voice)

ux-flow (independent)
    |
    +-- wireframe (can reference ux-flow)
```

**Recommendation**: Create brand-kit first. Other skills reference it for consistency.

---

## Design Output Location

All design outputs are saved to:
```
.waramity/design/
  brand-kit.md      # Brand identity document
  ux-flows/         # User flow diagrams
  wireframes/       # ASCII wireframes
  mockups/          # React/JSX components
  icons/            # SVG icon files
  ux-copy.md        # Microcopy guidelines
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Define brand identity | `/waramity` -> "create brand kit" |
| Map user journey | `/waramity` -> "ux flow for login" |
| Sketch layout | `/waramity` -> "wireframe home screen" |
| Design component | `/waramity` -> "ui mockup for button" |
| Create icons | `/waramity` -> "icon set for navigation" |
| Write microcopy | `/waramity` -> "ux copy for onboarding" |

---

## Rules

- Always check for existing brand-kit before ui-mockup, icon-set, or ux-copy
- If brand-kit doesn't exist and user requests dependent skill, suggest creating it first
- All outputs go to `.waramity/design/` directory
- Wireframes use ASCII art (text-based box drawing)
- UI mockups use React/JSX (SwiftUI-convertible syntax)
- If unclear which skill, ask: "Do you want to (1) define brand, (2) map flows, (3) sketch layout, (4) design components, (5) create icons, or (6) write copy?"
