---
name: app-solo-rank
description: >
  Use this skill when the user wants to find app ideas that are low-risk, require a small team,
  need no customer support, or are suitable for a solo developer.
  Triggers on: "low risk app", "solo developer app", "no support needed", "solo developer app",
  "app without support", "passive income app", "easiest app", "low risk app",
  "free server app", "app that doesn't need frequent updates", "lowest cost app".
  ALWAYS use this skill to produce a scored ranking widget with 5-axis analysis per app.
  Never return a plain text list — always build the interactive widget.
---

# App Solo Rank Skill

Renders a **solo-developer friendliness ranking widget** — scores each app on 5 axes and ranks by total.

## Scoring System (5 axes, 1–5 each, max 25)

| Axis | Label | 5 = Best | 1 = Worst |
|------|-------|----------|-----------|
| `team` | Small Team | 1 dev is enough | Requires 5+ people |
| `risk` | Risk Level | No license needed, no liability | Professional license required |
| `auto` | Automated | Fully automated, no human intervention | Requires human ops 24/7 |
| `cost` | Low Cost | Server $0/month, no inventory needed | Expensive infrastructure |
| `passive` | Passive | Build once, earn long-term | Requires active maintenance |

## Red Flags (score 1–2 on risk axis)
- Health apps giving medical advice → requires pharmacist/doctor certified content
- Legal advice apps → high liability if information is wrong
- Financial trading → requires SEC license
- Apps requiring real-time human moderation
- Apps handling sensitive personal data (medical records, financial accounts)

## Green Flags (score 4–5 on risk + auto axes)
- Pure calculation (tax, loan, BMI, date) → pure algorithms, no content liability
- Public domain data (public laws, calendars, government prices)
- User-generated content (UGC) → users create, developer doesn't
- Data updates ≤ once/year (tax rates, calendar rules)
- Offline-first → no server = no cost = no downtime

## Widget Structure

```
[Header: "Apps You Can Build Solo — Lowest Risk"]
[Criteria explanation row: 5 icons with short labels]
[Section title + color legend]
[App cards — sorted by total score desc]
```

### Per-App Card Layout

```
[Rank #N]  [App Name]  [Total Score /25] [⭐ if ≥23]
[One-line description]
[Score bars: 5 columns — label + colored bar + N/5]
[Why tags: key reasons as colored pills]
```

Score bar colors:
- 5/5 → `#34A853` (green)
- 3–4/5 → `#FBBC05` (yellow)
- 1–2/5 → `#E24B4A` (red)

Cards with score ≥ 23/25 get a gold highlight: `background:#EAF3DE; border-color:#97C459`

### Why Tags
Each app should have 4–5 short reason tags explaining its score. Examples:
- "Government data (public)"
- "Fixed algorithm calculation"
- "No database sync needed"
- "User-generated content (UGC)"
- "Updates once a year"
- "Server $0/month"
- "No liability"
- "Easy Freemium + ads"

## Interactivity

- **Click card** → `sendPrompt('Help analyze in detail for app "' + name + '" — tech stack, 30-day MVP, monetization, and steps to publish to App Store as solo developer')`
- Cards should show hover highlight

## Examples of Top-Scoring Apps (25/25)

- **Personal Income Tax Calculator**: pure math, public law, free data, updates once a year, no support needed
- **Auspicious Dates/Calendar App**: algorithm-only, no liability, passive ads, no database
- **Flashcard SRS Language App**: UGC, algorithm (Spaced Repetition), no content liability

## Styling

- Use CSS variables throughout
- Grid `repeat(auto-fit, minmax(260px,1fr))` for card layout on mobile
- Transition bars with 60ms setTimeout + CSS `transition: width .5s`
- Single `show_widget` call, no external dependencies
