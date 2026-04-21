---
name: business-plan
description: >
  Generate a comprehensive, investor-ready business plan document for any product, app, or startup idea.
  Use this skill whenever a user mentions: "business plan", "business planning", "business analysis",
  "market analysis", "revenue model", "monetization plan", "go-to-market", "GTM strategy", "MVP plan",
  "product roadmap", "startup plan", "app business plan", or when they describe a product idea and want
  to understand its commercial viability. Also trigger when the user says things like "is this a good idea?",
  "how do I make money from this?", or "help me plan my app/product/service".
  Output is always a structured .md file covering: market overview, product definition, tech stack,
  MVP roadmap, revenue model with projections, marketing plan, and risk analysis.
---

# Business Plan Skill

Produces a thorough, structured business plan document for any app, product, or service idea.
Output is always saved as a `.md` file and presented to the user.

---

## Trigger Contexts

Activate this skill for any of these requests:
- "make a business plan for [idea]"
- "business planning / business plan"
- "analyze this idea commercially"
- "how viable is [idea]?"
- "revenue model for [product]"
- "go-to-market strategy"
- User describes an app/product + wants planning, strategy, or monetization help

---

## Step 1 — Gather Context

Before writing, check the conversation for these. Ask only for what is missing:

| Info | Why Needed |
|---|---|
| Product/idea description | Core subject |
| Target market & geography | Sizing & pricing |
| Team size (solo / small / funded) | MVP scope & cost structure |
| Tech stack preferences | Build cost estimates |
| Monetization preference (B2C / B2B / SaaS / etc.) | Revenue model |
| Timeline expectations | MVP scope |
| Any known competitors | Positioning |

If the conversation already contains a detailed idea (like a spec doc or prior analysis), extract answers directly — do not ask again.

---

## Step 2 — Research (if web search is available)

Run these searches to ground the plan in real data:

```
[product category] market size [country/region] [year]
top [product category] apps [platform] [country]
[competitor names] revenue funding
[product category] app monetization benchmark
```

Use findings to populate market sizing and competitive landscape sections.
If web search is unavailable, use conservative estimates and label them clearly as estimates.

---

## Step 3 — Write the Business Plan

Follow the structure in `business-plan/template.md` exactly.
Fill every section — never leave a section as "TBD" or blank.

**Section order:**
1. Executive Summary (5–7 bullet KPIs)
2. Market Opportunity
3. Product Definition & Feature Set
4. Tech Stack & Architecture
5. MVP Roadmap (day-by-day or week-by-week)
6. Revenue Model with projections table
7. Cost Structure
8. Marketing & GTM Plan
9. Risk Analysis & Mitigation
10. Summary Scorecard

Read `business-plan/template.md` for the exact format, tables, and tone for each section.

---

## Step 4 — Quality Checks Before Saving

Before saving, verify:
- [ ] Every revenue figure has a clear assumption (e.g., "assuming 3% conversion of 10,000 users")
- [ ] MVP timeline is realistic for stated team size
- [ ] At least 3 risks identified with specific mitigations
- [ ] Marketing plan has at least one free/organic channel and one paid channel
- [ ] Competitive analysis names real competitors (not generic "existing solutions")
- [ ] All financial projections use consistent currency and timeframe

---

## Step 5 — Save & Present

Save to: `/mnt/user-data/outputs/[kebab-case-product-name]-business-plan.md`

Then call `present_files` with the output path.

End with a 2–3 sentence spoken summary of the biggest opportunity and the most important risk.

---

## Tone & Style Rules

- **Direct and confident.** No hedging language like "might" or "could potentially."
- **Numbers everywhere.** Every claim needs a number (market size, conversion rate, timeline, cost).
- **Opinionated tech stack.** Pick one recommended stack, explain why briefly. Don't list 5 alternatives.
- **Realistic projections.** Use conservative Year 1 estimates. Label Year 2+ as "target."
- **Local context:** If the product targets a specific region, use local currency, reference regional platforms, and local regulatory context where relevant.
- **Solo-developer friendly:** If team = 1 person, always emphasize no-server/serverless options, free tiers, and automation to reduce support burden.

---

## Reference Files

- `business-plan/template.md` — Full section-by-section template with example tables
- `business-plan/revenue-models.md` — Revenue model patterns (Freemium, SaaS, Marketplace, etc.) with benchmark conversion rates
- `business-plan/market-sizing.md` — How to estimate TAM/SAM/SOM for common product categories
