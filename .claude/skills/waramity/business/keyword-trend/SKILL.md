---
name: app-keyword-trend
description: >
  Use this skill whenever the user wants to see app ideas ranked in a Google Keyword Trends style
  interactive chart. Triggers on: "keyword trend app", "most searched apps", "rank app ideas",
  "popular apps", "Google Trends style", "rank apps", "most searched apps",
  "show app trends", "app idea keywords", "untapped app ideas" + any list/ranking request.
  ALWAYS use this skill when the user wants a visual bar chart ranking of multiple app ideas
  with search volume scores. Produces an interactive filterable widget — never plain text lists.
---

# App Keyword Trend Skill

Renders a **Google Trends-style interactive ranking widget** for app ideas in the Thai market.

## Widget Layout

```
[Google Trends brand bar]  •  Thailand YYYY  •  [score scale note]
[Stats row: total / 🔥🔥🔥 count / 🔥🔥 count / categories]
[Category filter tabs — color per category]
[Search input]
[Table header: # | App/Keyword | Search Volume Bar | Score | Trend]
[Animated rows — bar fills on load]
[Footer: data source]
```

## Row Data Fields

| Field | Type | Description |
|-------|------|-------------|
| `n` | string | App name e.g. "TOEIC App" |
| `cat` | string | Category ID |
| `s` | number | Score 0–100 (100 = highest demand) |
| `t` | 'up'/'dn'/'fl' | Trend direction ▲ ▼ — |
| `opp` | 1/2/3 | Opportunity: 3=🔥🔥🔥 2=🔥🔥 1=🔥 |
| `color` | hex | Bar color (use category color) |
| `tag` | string | Short category label |
| `tagBg/tagFg` | hex | Pill background / text color |
| `detail` | string | One-line description |

## Scoring Guide (Thai Market)

- **95–100**: Mainstream + no good solution (tax, TOEIC, market prices)
- **80–94**: Large niche — college admissions, home repair, labor law
- **60–79**: Solid niche — secondhand, pets, auspicious dates
- **40–59**: Community/hobby — temples, music, rehearsal studios
- **20–39**: Very niche — exotic pets, hiking
- **< 20**: Fan/celeb — very small market

## Opportunity Score

```
🔥🔥🔥 = High search + no Thai solution + no complex licensing
🔥🔥   = Good demand + some competition or moderate complexity
🔥     = Niche / high risk / already solved
```

## Category Colors

```
Finance/Tax    #185FA5  bg:#E6F1FB  fg:#0C447C
Health/Medical #34A853  bg:#EAF3DE  fg:#27500A
Shopping/Deals #D4537E  bg:#FBEAF0  fg:#72243E
Food/Agriculture #D85A30  bg:#FAECE7  fg:#712B13
Entertainment/Games #534AB7  bg:#EEEDFE  fg:#3C3489
Auto/Travel    #854F0B  bg:#FAEEDA  fg:#633806
Lifestyle      #993556  bg:#FBEAF0  fg:#72243E
Government/Legal #607D8B  bg:#ECEFF1  fg:#37474F
Other          #5F5E5A  bg:#F1EFE8  fg:#444441
```

## Interactivity (required)

- **Category tabs**: filter rows; tab gets category color when `.on`
- **Search box**: real-time filter on `n` and `tag`
- **Animated bars**: setTimeout 60ms → transition width 0% → s% (0.55s ease)
- **Click row** → `sendPrompt('Help analyze app idea "' + name + '": ' + detail + ' — what is the business opportunity, monetization strategy, and MVP roadmap?')`
- Optional top sparkline cards for top 5

## Styling

Use CSS variables: `--color-background-*`, `--color-border-*`, `--color-text-*`, `--font-sans`, `--border-radius-md/lg`

All in a single `show_widget` call. No external CSS files.
