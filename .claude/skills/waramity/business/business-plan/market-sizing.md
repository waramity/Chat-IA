# Market Sizing Guide

How to estimate TAM / SAM / SOM for common product categories.
Always cite your source or label estimates clearly.

---

## Framework

**TAM** (Total Addressable Market): Everyone who could theoretically use this.
**SAM** (Serviceable Addressable Market): People you can realistically reach with your distribution.
**SOM** (Serviceable Obtainable Market): Realistic capture in Year 1–2.

Rule of thumb for solo developer SOM: 0.1–1% of SAM in Year 1.

---

## Thailand Market Reference Data (2025 estimates)

| Segment | Size | Notes |
|---|---|---|
| Internet users | ~63M | |
| Smartphone users | ~55M | |
| iOS users | ~10M | ~18% of smartphone users |
| Personal income tax filers | ~11M | Revenue Department |
| Salaried employees (formal) | ~18M | SSO data |
| Freelancers / gig workers | ~5M | estimated |
| SME businesses | ~3.1M | BOT data |
| Active investors (SET/funds) | ~4M | SEC data |
| E-commerce buyers (active) | ~25M | ETDA |
| App Store spenders (any paid) | ~2M | estimated |

---

## Common App Categories — Sizing Approach

### Finance / Tax Apps
- SAM: Tax filers who own a smartphone = ~9M
- Further narrow by: iOS only = ~1.6M
- SOM Year 1: 0.1–0.5% = 1,600–8,000 downloads
- Notes: Strong seasonal spike Jan–Mar (tax filing season)

### Health & Fitness
- SAM: Smartphone users interested in health = ~15M
- iOS SAM: ~2.7M
- SOM Year 1: 0.05–0.2% = 1,350–5,400
- Notes: Very competitive category

### Productivity / Tools
- SAM: Knowledge workers with smartphone = ~8M
- iOS SAM: ~1.4M
- SOM Year 1: 0.1–0.5% = 1,400–7,000

### Food / Restaurant
- SAM: People who order food online = ~20M
- Notes: Dominated by GrabFood/FoodPanda. Niche only.

### Education / Learning
- SAM: Students + working adults seeking skills = ~20M
- iOS SAM: ~3.6M
- SOM Year 1: 0.05–0.2%

---

## How to Size a Market You Don't Know

1. **Top-down**: Find total market size from research report, then apply penetration % 
   Example: "5M freelancers × 20% use financial tools × 10% would pay = 100K TAM for paid tier"

2. **Bottom-up**: Build from unit economics
   Example: "If I get 10 downloads/day from ASO, that's 3,650/year at my current ranking"

3. **Analog**: Find a similar app in a similar market and estimate their numbers
   Example: "iTAX claims 1M+ downloads over 5 years in Thailand → annual run rate ~200K"

---

## Idea Scoring Criteria (for Scorecard section)

Score each 1–5:

**Market Size** (1 = tiny niche, 5 = mass market)
- 5: >5M reachable users in Thailand
- 4: 1–5M reachable users
- 3: 500K–1M
- 2: 100K–500K
- 1: <100K

**Technical Complexity** (1 = very complex, 5 = simple)
- 5: Pure logic/calculations, no backend, no ML
- 4: Simple CRUD + local storage
- 3: Backend needed, standard patterns
- 2: Real-time sync, complex integrations
- 1: ML, blockchain, real-time multiplayer, hardware

**Monetization Clarity** (1 = unclear, 5 = obvious)
- 5: Users clearly willing to pay, comparable apps exist at price point
- 4: Strong paid signal, minor friction
- 3: Some paid signal, needs testing
- 2: Mostly free market, ad-dependent
- 1: No clear monetization path

**Maintenance Burden** (1 = very high, 5 = very low)
- 5: Offline, data changes <1x/year, no user support needed
- 4: Minimal server, infrequent updates required
- 3: Monthly updates, some support needed
- 2: Frequent data updates, active moderation
- 1: Real-time data feeds, high support volume

**Time to First Revenue** (1 = slow, 5 = fast)
- 5: Revenue possible within 30 days of launch
- 4: 30–60 days
- 3: 60–90 days
- 2: 3–6 months
- 1: 6+ months
