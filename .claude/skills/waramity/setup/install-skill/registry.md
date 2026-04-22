# Skill Registry

Available skills that can be installed via `install-skill`.

| ID | Name | Description | Repo | Connects To |
|----|------|-------------|------|-------------|
| 1 | ui-ux-pro-max-skill | Design intelligence for building professional UI/UX | `nextlevelbuilder/ui-ux-pro-max-skill` | design |
| 2 | swift-ios-skills | iOS 26+, Swift 6.3, SwiftUI, Apple frameworks | `dpearson2699/swift-ios-skills` | dev |
| 3 | gemma4-ios-skill | Google Gemma 4 on-device with LiteRT-LM | `waramity/gemma4-ios-skill` | dev |
| 4 | andrej-karpathy-skills | LLM coding best practices from Andrej Karpathy | `forrestchang/andrej-karpathy-skills` | dev |
| 5 | rtk | CLI proxy reducing LLM tokens 60-90% on dev commands | `rtk-ai/rtk` | dev |

---

## Adding New Skills

To add a skill to the registry:

1. Add a new row to the table above
2. Use the next available ID
3. Set `Connects To` based on skill category:
   - `design` — UI/UX, branding, visual design
   - `dev` — Development, coding, tooling
   - `business` — Business planning, analysis

## Format

```
| {ID} | {skill-name} | {short description} | `{owner}/{repo}` | {category} |
```
