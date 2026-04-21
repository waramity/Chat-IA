---
name: ux-copy
description: >
  Create microcopy and UI text guidelines.
  Reads brand-kit for voice and tone.
  Triggers on: "ux copy", "microcopy", "button text", "error messages", "onboarding text", "empty state".
---

# Skill: ux-copy

Create microcopy and UI text that matches brand voice.

## Workflow

### 1. Check Brand Kit

```bash
# Check if brand-kit exists
cat .waramity/design/brand-kit.md 2>/dev/null
```

If brand-kit exists, extract:
- Brand voice (formal/casual/friendly)
- Tone guidelines
- Voice examples

If no brand-kit:
> "No brand-kit found. What tone should I use?"
> - **Professional** - Clear, direct, minimal personality
> - **Friendly** - Warm, approachable, conversational
> - **Playful** - Fun, witty, uses personality
> - **Minimal** - Ultra-concise, no fluff

### 2. Gather Context

Ask:
> "What copy do you need?"
> - **Buttons/CTAs** - Action text for buttons
> - **Error messages** - Validation, API errors, failures
> - **Empty states** - No data, first-time user
> - **Onboarding** - Welcome, tutorials, tooltips
> - **Notifications** - Push, in-app, alerts
> - **Form labels** - Input labels, placeholders, hints
> - **Full screen** - All copy for a specific screen

### 3. Generate Copy

Create copy following UX writing principles:

**Button/CTA Copy:**
```markdown
## Buttons & CTAs

| Action | Copy | Avoid |
|--------|------|-------|
| Sign up | Get Started | Submit |
| Login | Sign In | Login to your account |
| Save | Save Changes | Click to Save |
| Delete | Delete | Remove Item |
| Cancel | Cancel | Go Back |
| Confirm | Confirm | Yes |
| Primary CTA | [Action] + [Benefit] | Click Here |

### Examples
- "Start Free Trial" (not "Submit")
- "Create Account" (not "Register")
- "Save Changes" (not "Update")
```

**Error Messages:**
```markdown
## Error Messages

### Formula
**What happened** + **Why** + **How to fix**

| Error Type | Message |
|------------|---------|
| Empty field | "Please enter your email" |
| Invalid format | "Enter a valid email (e.g., name@example.com)" |
| Network error | "Couldn't connect. Check your internet and try again." |
| Server error | "Something went wrong. We're working on it." |
| Not found | "We couldn't find what you're looking for." |
| Permission | "You don't have access to this. Contact your admin." |

### Avoid
- Technical jargon ("Error 500", "null reference")
- Blame language ("You entered wrong password")
- Vague messages ("An error occurred")
```

**Empty States:**
```markdown
## Empty States

### Formula
**Acknowledge** + **Explain benefit** + **CTA**

| State | Headline | Body | CTA |
|-------|----------|------|-----|
| No items | "No [items] yet" | "When you [action], they'll show up here." | "[Add first item]" |
| No results | "No results for '[query]'" | "Try different keywords or filters." | "Clear filters" |
| First time | "Welcome!" | "[Benefit of feature]. Get started by..." | "[First action]" |

### Examples
- **No messages**: "No messages yet. When someone reaches out, you'll see it here."
- **No search results**: "No results for 'xyz'. Try a different search term."
- **Empty cart**: "Your cart is empty. Find something you'll love."
```

**Onboarding Copy:**
```markdown
## Onboarding

### Welcome Screen
- Headline: [Value proposition in 5-8 words]
- Body: [1-2 sentences expanding on benefit]
- CTA: [Get Started / Let's Go / Continue]

### Feature Tours
| Step | Headline | Body |
|------|----------|------|
| 1 | [Feature name] | [What it does in <15 words] |
| 2 | [Feature name] | [What it does in <15 words] |

### Tooltips
- Keep under 10 words
- Focus on "how" not "what"
- Example: "Tap to add a new item"
```

**Notifications:**
```markdown
## Notifications

### Push Notifications
| Type | Title | Body |
|------|-------|------|
| Reminder | "[Action] reminder" | "Don't forget to [action]" |
| Update | "[Feature] is ready" | "[Benefit or what's new]" |
| Social | "[Name] [action]" | "[Preview or context]" |

### In-App Alerts
| Type | Message |
|------|---------|
| Success | "Done! [What happened]" |
| Info | "[What to know]" |
| Warning | "[What might happen]. [Suggestion]" |
```

### 4. Create Copy Guide

Compile into comprehensive document:

```markdown
# UX Copy Guide: [Project Name]

Generated: YYYY-MM-DD
Voice: [From brand-kit or specified]
Tone: [Description]

## Voice Principles

1. **[Principle]** - [Example]
2. **[Principle]** - [Example]
3. **[Principle]** - [Example]

## Word List

| Use | Don't Use |
|-----|-----------|
| Sign in | Login |
| Get started | Submit |
| Something went wrong | Error occurred |

## Copy by Category

### Buttons & CTAs
[Table]

### Error Messages
[Table]

### Empty States
[Table]

### Onboarding
[Table]

### Notifications
[Table]

## Capitalization

- **Buttons**: Sentence case ("Get started")
- **Headings**: Title Case ("Your Account Settings")
- **Body**: Sentence case

## Punctuation

- No periods on buttons
- No periods on headings
- Use periods in body text and descriptions
```

### 5. Save & Present

```bash
# Ensure directory exists
mkdir -p .waramity/design

# Save copy guide
# File: .waramity/design/ux-copy.md
```

---

## UX Writing Principles

| Principle | Do | Don't |
|-----------|-----|-------|
| Clear | "Save changes" | "Submit form data" |
| Concise | "Email required" | "The email field is required" |
| Useful | "Enter 8+ characters" | "Invalid password" |
| Human | "Something went wrong" | "Error 500" |
| Actionable | "Try again" | "OK" |

---

## Rules

- Always check brand-kit for voice guidelines
- Keep microcopy under 15 words where possible
- Lead with verbs for actions
- Be specific about errors and how to fix them
- Test copy at actual font sizes (long copy might truncate)
- Avoid jargon, technical terms, and insider language
