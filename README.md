<div align="center">


![Gemini_Generated_Image_gou91kgou91kgou9](https://github.com/user-attachments/assets/f2fa1995-81a7-431b-b453-02d406b75864)


# 🏛️ design-system-architect

**A Claude Code skill that thinks like a Design Systems Lead, acts like a Senior Product Designer, and executes like a Frontend Architect.**

[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-8B5CF6?style=flat-square&logo=anthropic&logoColor=white)](https://claude.ai/code)
[![Figma MCP](https://img.shields.io/badge/Figma_MCP-Required-F24E1E?style=flat-square&logo=figma&logoColor=white)](https://www.figma.com)
[![shadcn/ui](https://img.shields.io/badge/shadcn%2Fui-Aligned-000000?style=flat-square)](https://ui.shadcn.com)
[![WCAG 2.2](https://img.shields.io/badge/WCAG_2.2-AA%2FAAA-1A9C3E?style=flat-square)](https://www.w3.org/WAI/WCAG22/quickref/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

*Drop a Figma URL. Get a full system audit, a prioritized action plan, and direct Figma execution — in one conversation.*

</div>

---

## What it is

`design-system-architect` is a [Claude Code skill](https://claude.ai/code) that embeds a Design System Architect directly into your workflow. It connects to your Figma file via MCP, audits the entire system, surfaces every gap with evidence, and — after your approval — applies fixes directly back into Figma.

No back-and-forth. No vague recommendations. Every finding includes a concrete fix. Every fix is token-ordered, regression-safe, and composability-preserving.

---

## How it works

The skill runs a strict **3-phase workflow**:

```
Phase 1 ──── AUDIT ──────────────── Read-only. Never touches the file.
Phase 2 ──── REPORT + PLAN ──────── Dual output: human report + machine JSON.
Phase 3 ──── EXECUTION ──────────── Only runs after your explicit approval.
```

### Phase 1 — Audit

Parses the entire Figma structure programmatically:

- All pages, component sets, variants, and standalone components
- Variable collections (primitive → semantic → component token hierarchy)
- Token scopes, mode support, and alias chains
- Text styles and effect styles
- Naming consistency and governance readiness

Flags every inconsistency, gap, and anti-pattern with **evidence from the actual Figma data** — not assumptions.

---

### Phase 2 — Dual Report

Two outputs, produced together:

#### Output A — Human Report

```
Executive Summary
├── System maturity score (0–5)
├── shadcn/ui alignment: low / medium / high
├── Key risks
├── Scalability blockers
└── Top 5 critical issues

Critical Gaps
└── Each gap: what it is · why it matters · evidence from audit

Action Plan (prioritized)
└── Ordered by lowest effort + highest impact
    └── Each item: change · why · impact · effort · risk
```

#### Output B — Machine JSON

```json
{
  "system_maturity": 3.5,
  "shadcn_alignment": "high",
  "issues": [
    {
      "id": "GAP-01",
      "title": "Mode collection: ALL_SCOPES on all 72 semantic tokens",
      "category": "tokens",
      "severity": "critical",
      "impact": "Every Figma property picker flooded with all tokens simultaneously",
      "evidence": "3. Mode collection: 72/72 variables have scopes: ['ALL_SCOPES']",
      "fix": "Remap to context-specific scopes: backgrounds → FRAME_FILL/SHAPE_FILL, text → TEXT_FILL, borders → STROKE_COLOR"
    }
  ],
  "missing_inventory": {
    "components": ["Toast", "Tabs/List container"],
    "variants": ["Button: State=Pressed for Size=default/sm/lg"],
    "tokens": ["motion/duration/*", "motion/ease/*", "z-index/*"],
    "states": ["Checkbox: Indeterminate"],
    "documentation": []
  },
  "action_plan": [
    {
      "step": 1,
      "title": "Fix spacing token scopes",
      "description": "Extend all spacing/* scopes from ['GAP'] to ['GAP','WIDTH_HEIGHT']",
      "impact": "Unlocks padding/width tokenization system-wide",
      "effort": "low",
      "dependencies": []
    }
  ],
  "implementation_specs": {
    "tokens": {},
    "components": {},
    "storybook_mapping": {}
  }
}
```

---

### Phase 3 — Execution

After you approve (all or specific steps), the skill applies changes directly in Figma:

| Execution order | Why |
|---|---|
| 1. Tokens / variables | Foundation — everything else depends on this |
| 2. Foundations (color, type, spacing) | Resolve before components reference them |
| 3. Components | Bind to correct tokens, add missing variants/states |
| 4. Documentation / governance | Lock in naming and contribution rules |

**Guarantees:** backward compatibility, no visual regressions, no renames without migration path.

---

## Audit scope

<table>
<tr><th>Area</th><th>What's analyzed</th></tr>
<tr>
<td><strong>🎨 Tokens</strong></td>
<td>Primitive → semantic → component hierarchy · naming consistency · missing categories (motion, z-index, opacity) · hardcoded values · variable scopes · alias chains</td>
</tr>
<tr>
<td><strong>🌈 Color</strong></td>
<td>Primitive palette · semantic token mapping · state colors · dark mode coverage · ALL_SCOPES pollution · hardcoded hex</td>
</tr>
<tr>
<td><strong>📐 Typography</strong></td>
<td>Type scale completeness · line-height rhythm · text style coverage · font weight tokenization · letter spacing</td>
</tr>
<tr>
<td><strong>🧩 Components</strong></td>
<td>Coverage vs. shadcn/ui baseline · variant completeness · state coverage (hover/focus/error/loading/disabled/empty) · composability · redundancy</td>
</tr>
<tr>
<td><strong>♿ Accessibility</strong></td>
<td>WCAG 2.2 AA/AAA contrast · focus visibility · touch targets (48×48px min) · keyboard navigation · semantic clarity</td>
</tr>
<tr>
<td><strong>📚 Storybook</strong></td>
<td>Variant → props mapping · token usage in code · controls definition · dev-ready API clarity</td>
</tr>
<tr>
<td><strong>📄 Documentation</strong></td>
<td>Naming conventions · usage guidelines · Do/Don't patterns · contribution readiness</td>
</tr>
</table>

---

## Real-world results

Tested on **Apollo v2.1** — a production shadcn/ui-based design system:

| Metric | Before | After |
|---|---|---|
| System maturity | 3.5 / 5 | 4.5 / 5 |
| Total variables | 883 | 901 |
| Button variants | 138 | 156 |
| Input variants | 28 | 36 |
| Checkbox variants | 8 | 12 |
| Token scopes polluting all pickers | 72 (`ALL_SCOPES`) | 0 |
| Hardcoded cornerRadius on Button | ✗ | ✓ bound to token |
| Motion tokens | 0 | 11 (duration + easing) |
| Z-index tokens | 0 | 7 |
| Spacing scope coverage | gap only | gap + width/height |

**11 issues found · 10 action plan steps executed · 0 visual regressions**

---

## Example triggers

```
"audit my design system: figma.com/design/abc123/MyDS"
"our tokens are a mess — some colors are hardcoded and not using variables"
"devs can't find the right component in our Figma library"
"what are we missing compared to the shadcn baseline?"
"make our Figma dev-ready before we onboard new engineers"
"our spacing tokens only show up in gap pickers, not padding — is that a scope issue?"
"wcag audit our component library: figma.com/design/..."
"I want a full maturity assessment of our design system"
```

---

## Requirements

| Requirement | Notes |
|---|---|
| [Claude Code](https://claude.ai/code) | CLI or desktop app |
| Figma plugin for Claude Code | Install via `/plugin` → figma in Claude Code |
| Figma file with edit access | The skill reads and writes to the file |
| React + Tailwind project (optional) | Default stack assumption; can be overridden |

---

## Installation

```bash
# Clone the skill
git clone https://github.com/lexsanchez-g2/ds-architect.git

# Copy SKILL.md to Claude Code's personal skills directory
mkdir -p ~/.claude/skills/design-system-architect
cp ds-architect/SKILL.md ~/.claude/skills/design-system-architect/SKILL.md
```

Then in Claude Code, run `/reload-plugins` — the skill appears automatically in the available skills list.

---

## Design principles

The skill is built around [shadcn/ui](https://ui.shadcn.com/) architecture conventions:

- **Composition over monoliths** — every component must be slot-based and decomposable
- **Token-first** — tokens before components, components before patterns
- **No assumptions without evidence** — every issue cites data from the Figma audit
- **Every issue gets a fix** — identification without remediation is noise
- **System over local** — prefer root-cause fixes over patches

---

## License

MIT © [Lex Sanchez](https://github.com/lexsanchez-g2)
