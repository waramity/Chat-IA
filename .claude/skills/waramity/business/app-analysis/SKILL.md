---
name: app-business-analysis
description: >
  Use this skill whenever the user wants a deep business analysis for a SINGLE app idea.
  Triggers on: "analyze app idea", "make app business plan", "detailed analysis",
  "tech stack for app", "MVP roadmap app", "how to monetize app", "publish to App Store",
  "app business opportunity", "help analyze app [name]", or when a user clicks through from
  a keyword trend chart and asks for full analysis of one specific app.
  ALWAYS use this skill for single-app deep dives. Produces a multi-tab interactive widget
  with live demo, market data, tech stack, MVP, revenue model, risk, and growth chart.
---

# App Business Analysis Skill

Renders a **multi-tab interactive business analysis widget** for one app idea.

## Required Tabs

Build a tabbed widget. Show/hide panes with `.on` class via JS click handler.

---

### Tab 1: Live Demo (if feasible)
If the app has a computable function (tax, loan payment, BMI, unit converter, date calculator):
- Build a **working calculator** inside the widget using real formulas
- Real-time output as user types (oninput)
- End with action button: `sendPrompt('Help write code for [app] ...')`

If no calculable function → skip this tab or replace with "Mock UI" screenshot-style mockup.

---

### Tab 2: Business Opportunity

**KPI cards** (4–5, grid auto-fit minmax 110px):
- Market size (e.g. "11M+ taxpayers")
- Keyword score (from trend ranking)
- Competitor count ("~0 competitors with good UX")
- Estimated Year-2 revenue

**2-column cards:**
- Left: Pain Points — red dots + problems users face today
- Right: Why We Win — green dots + unfair advantages

**User Segments:** bullet list of 3–4 target personas

---

### Tab 3: Tech Stack

Grid of stack cards (auto-fit minmax 140px). Each card:
```
[category label]  e.g. "Framework"
[tool name]       e.g. "React Native"
[why]             1–2 lines reasoning
[badge]           "Recommended" / "Free" / "$0/month"
```

Always include:
- Frontend framework (React Native or Flutter for mobile)
- State management (Zustand preferred for simplicity)
- Storage (AsyncStorage / MMKV if offline-first)
- Backend needs (often "not needed" for calculator apps)
- Analytics (Firebase free tier)
- Payments (RevenueCat for IAP)
- Build tool (EAS Build — no Mac needed)

Show **monthly infrastructure cost** prominently. For solo offline apps: $0.

---

### Tab 4: MVP Roadmap

Vertical timeline with connector line (::before pseudo-element):

```
[Phase badge] [Week/Day range]
[Title]
[Detail — what to build, what to test]
[Tech tags]
[Target metric badge]
```

Typical phases:
1. Setup + Core Logic (days 1–3): algorithms, unit tests, no UI yet
2. Main UI (days 4–10): 2–3 key screens, test on real device daily
3. Content + Polish (days 11–17): onboarding, tips, dark mode, user testing x5
4. Store Assets (days 18–22): icon, screenshots, privacy policy, developer accounts
5. Build + Submit (days 23–28): EAS build, App Store + Play Store submission
6. Launch (days 29–30): Pantip, Facebook group, TikTok clip, Firebase tracking

Show Apple ($99/yr) + Google Play ($25) = **$124 total first cost**.

---

### Tab 5: Revenue Streams

For each stream, show as a card row:
```
[icon] [stream name + description] [annual estimate Year 2]
```

Common revenue streams:
| Stream | Typical Price | Notes |
|--------|--------------|-------|
| Freemium B2C | $1–3/mo | Basic free, unlock premium |
| Annual B2C | $10–30/yr | 40–50% discount vs monthly |
| B2B SaaS | $15–100/mo | SME / HR / clinics |
| Affiliate | 0.5–2% | Fund houses, insurance, pharma |
| Gov / White-label | $15K–150K | Project-based |
| In-app purchase | Variable | Games, content |

Show **total Year-2 revenue estimate** at bottom with margin note.

---

### Tab 6: Publish to Store

Numbered steps with cost badges:
1. Apple Developer — $99/yr
2. Google Play — $25 once
3. Privacy Policy — GitHub Pages → Free
4. Store Listing — title (include year + keyword), description 4000 chars, screenshots, icon 1024px
5. Build — `eas build --platform all` → free 30 builds/mo
6. Submit — Android 1–3 days, iOS 1–7 days
7. ASO — Thai + English keywords list

---

### Tab 7: Risk & Opportunity

2×2 grid:
- 🔴 Risk A: data accuracy / liability + mitigation
- 🔴 Risk B: legal / license requirement + mitigation
- 🟢 Opportunity A: government partnership / API access
- 🟢 Opportunity B: international expansion (Myanmar/English)

---

## Revenue/Growth Chart (below tabs)

Chart.js line chart, 18–24 month projection:
```js
// CDN
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>

datasets: [
  { label: 'MAU', yAxisID: 'y1', borderColor: '#639922' },         // left axis (thousands)
  { label: 'B2C Rev', yAxisID: 'y2', borderDash: [5,3] },          // right axis ($K)
  { label: 'B2B Rev', yAxisID: 'y2', borderDash: [3,2] }
]
```
No legend shown in chart — show custom legend above.

---

## Action Buttons (required at bottom)

Always include these `sendPrompt` buttons:
```
[Write PRD ↗]  [Tech Stack / Code ↗]  [Pitch Deck ↗]  [Go-to-Market ↗]  [Apply for Funding ↗]
```

---

## Thai Market Benchmarks

Use these when estimating revenue:
- B2C conversion from free → paid: 3–5% typical
- Freemium at $1–3/mo: mass market (impulse-buy)
- Annual $10–30/yr: 30–40% choose this over monthly
- SME B2B $30/mo: realistic for 500–2,000 customers
- D7 retention target for utility apps: 40%+
- First-week downloads target after Pantip + Facebook launch: 1,000+

## Styling

- CSS variables for all colors (dark/light compatible)
- Tabs: `.tab.on` with `border-bottom: 2px solid #378ADD`
- Cards: `border: .5px solid var(--color-border-tertiary)`
- All in single `show_widget` call
