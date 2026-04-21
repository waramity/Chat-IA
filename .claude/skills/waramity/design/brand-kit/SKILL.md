---
name: brand-kit
description: >
  Define brand identity including colors, typography, voice, and logo direction.
  Foundation skill - other design skills reference this.
  Triggers on: "brand kit", "define brand", "color palette", "typography", "brand voice", "logo direction".
---

# Skill: brand-kit

Create or update the project's brand identity document.

## Workflow

### 1. Check Existing Brand Kit

```bash
# Check if brand-kit already exists
ls .waramity/design/brand-kit.md 2>/dev/null
```

If exists, ask:
> "Found existing brand-kit. Do you want to:
> 1. **Update** - Modify specific sections
> 2. **Replace** - Start fresh
> 3. **View** - Show current brand-kit"

### 2. Gather Context

Ask focused questions with pre-selected options:

**Project Type:**
> "What type of project is this?"
> - Mobile app (iOS/Android)
> - Web application
> - SaaS product
> - E-commerce
> - Portfolio/Personal
> - Other: ___

**Brand Personality:**
> "Select up to 3 personality traits:"
> - Professional / Corporate
> - Playful / Fun
> - Minimal / Clean
> - Bold / Energetic
> - Warm / Friendly
> - Luxurious / Premium
> - Technical / Modern
> - Organic / Natural

**Color Preference:**
> "Any color direction?"
> - Specific colors in mind (provide hex/names)
> - Mood-based (calm, energetic, trustworthy, etc.)
> - Industry-standard (fintech blue, health green, etc.)
> - No preference - suggest based on personality

### 3. Generate Brand Kit

Create the brand-kit document with these sections:

```markdown
# Brand Kit: [Project Name]

Generated: YYYY-MM-DD

## Brand Personality

**Core Traits:** [Selected traits]
**Voice:** [Formal/Casual/Friendly/Professional]
**Tone:** [Description of how the brand speaks]

## Color Palette

### Primary Colors
| Name | Hex | RGB | Usage |
|------|-----|-----|-------|
| Primary | #XXXXXX | rgb(X,X,X) | Main actions, headers |
| Secondary | #XXXXXX | rgb(X,X,X) | Supporting elements |

### Neutral Colors
| Name | Hex | RGB | Usage |
|------|-----|-----|-------|
| Background | #XXXXXX | rgb(X,X,X) | Page backgrounds |
| Surface | #XXXXXX | rgb(X,X,X) | Cards, containers |
| Text Primary | #XXXXXX | rgb(X,X,X) | Main text |
| Text Secondary | #XXXXXX | rgb(X,X,X) | Supporting text |
| Border | #XXXXXX | rgb(X,X,X) | Dividers, outlines |

### Semantic Colors
| Name | Hex | Usage |
|------|-----|-------|
| Success | #XXXXXX | Confirmations, success states |
| Warning | #XXXXXX | Cautions, pending states |
| Error | #XXXXXX | Errors, destructive actions |
| Info | #XXXXXX | Information, tips |

## Typography

### Font Stack
- **Headings:** [Font name] (or system: -apple-system, SF Pro)
- **Body:** [Font name]
- **Mono:** [Font name] (for code)

### Type Scale
| Level | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| H1 | 32px | Bold | 1.2 | Page titles |
| H2 | 24px | Semibold | 1.3 | Section headers |
| H3 | 20px | Semibold | 1.4 | Subsections |
| Body | 16px | Regular | 1.5 | Main content |
| Small | 14px | Regular | 1.4 | Captions, hints |
| Micro | 12px | Medium | 1.3 | Labels, badges |

## Spacing System

Base unit: 4px

| Token | Value | Usage |
|-------|-------|-------|
| xs | 4px | Tight spacing |
| sm | 8px | Related elements |
| md | 16px | Standard spacing |
| lg | 24px | Section spacing |
| xl | 32px | Large gaps |
| 2xl | 48px | Page sections |

## Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| none | 0px | Sharp corners |
| sm | 4px | Subtle rounding |
| md | 8px | Standard rounding |
| lg | 16px | Cards, modals |
| full | 9999px | Pills, avatars |

## Logo Direction

**Style:** [Wordmark / Symbol / Combination]
**Characteristics:** [Description]
**Do's:** [Guidelines]
**Don'ts:** [Guidelines]

## Brand Voice Examples

| Context | Example |
|---------|---------|
| Success message | "[Example text]" |
| Error message | "[Example text]" |
| Empty state | "[Example text]" |
| CTA button | "[Example text]" |
```

### 4. Quality Check

Verify:
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] Palette has sufficient neutral colors
- [ ] Typography scale is consistent
- [ ] Voice examples match personality

### 5. Save & Present

```bash
# Ensure directory exists
mkdir -p .waramity/design

# Save brand-kit
# File: .waramity/design/brand-kit.md
```

Show user:
- Summary of brand identity
- Color palette preview (if terminal supports)
- Next steps: "Brand kit saved. You can now create ui-mockup, icon-set, or ux-copy."

---

## Output Format

Single markdown file at `.waramity/design/brand-kit.md`

---

## Rules

- Always check for existing brand-kit first
- Ask clarifying questions before generating
- Ensure color contrast accessibility
- Provide concrete examples, not abstract descriptions
- This is the foundation - suggest creating it before other design skills
