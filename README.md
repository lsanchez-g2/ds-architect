# design-system-architect

A Claude Code skill that acts as an embedded **Design System Architect** — auditing Figma design systems, producing structured reports, and executing approved fixes directly in Figma via MCP.

## What it does

Drop any Figma design system URL in the chat and ask for an audit. The skill runs a three-phase workflow automatically:

**Phase 1 — Audit (read-only)**  
Parses the full Figma structure: pages, components, variants, variables, text styles, and effect styles. Detects inconsistencies, missing tokens, broken composability, accessibility gaps, and Storybook alignment issues. No modifications made.

**Phase 2 — Dual Report**  
Produces two outputs simultaneously:
- **Human report** — executive summary (maturity score 0–5, shadcn-ui alignment), critical gaps with evidence, and a prioritized action plan
- **Machine JSON** — structured audit output with severity, evidence, and fix for every issue; missing inventory; and Storybook-ready component API specs

**Phase 3 — Execution (after approval)**  
Applies changes directly in Figma using the Figma MCP. Token scopes, variable bindings, component variants, new tokens — all executed in the correct dependency order with no visual regressions.

## Requirements

- [Claude Code](https://claude.ai/code) with the [Figma MCP plugin](https://www.figma.com/community/plugin/claude-code) installed
- The `figma` Claude Code plugin (`/plugin` → figma)
- A Figma file with edit access

## Design system philosophy

Built around [shadcn-ui](https://ui.shadcn.com/) principles:
- Token-driven styling (primitive → semantic → component hierarchy)
- Variant-based component APIs
- Tailwind + design token alignment
- Storybook-controllable components
- WCAG 2.2 AA/AAA accessibility compliance

## Audit scope

| Area | What's checked |
|---|---|
| **Tokens** | Hierarchy, naming, scopes, missing categories (motion, z-index, opacity) |
| **Colors** | Primitive vs. semantic vs. mode tokens, dark mode, hardcoded values |
| **Typography** | Scale, rhythm, line-height, text style coverage |
| **Components** | Coverage vs. shadcn-ui baseline, variant completeness, state coverage |
| **Accessibility** | WCAG contrast, focus visibility, touch targets, semantic clarity |
| **Composability** | Slot-based patterns, fragmentation, redundancy |
| **Documentation** | Naming conventions, usage guidelines, contribution readiness |
| **Storybook** | Variant → props mapping, controls definition, dev-ready API |

## Example triggers

```
"audit my design system: figma.com/design/abc123/MyDS"
"our tokens are a mess — some colors are hardcoded and not using variables"
"devs can't find the right component in our Figma library"
"what are we missing compared to the shadcn baseline?"
"make our Figma dev-ready before we onboard new engineers"
"our spacing tokens only show up in gap pickers, not padding — is that a scope issue?"
```

## Tested on

Apollo v2.1 design system — a production shadcn-based system with 90 pages, 883 variables, 168 text styles, and 45 components. Audit identified 11 issues; all 10 approved fixes were applied in Figma with 0 visual regressions.

## Installation

1. Clone this repo
2. Copy `SKILL.md` to `~/.claude/skills/design-system-architect/SKILL.md`
3. Reload Claude Code plugins (`/reload-plugins`)

The skill appears automatically in Claude Code's available skills list.

## License

MIT
