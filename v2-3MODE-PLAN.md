# design-system-architect v2.0 — 3-Mode Architecture

## Overview

Evolving the skill from single-mode (Hardcore only) to 3 modes: **Hardcore**, **Soft**, and **Spec**.

---

## Phase 0 — Mode Selection (NEW)

When invoked, present:

```
Which mode do you need?

🔥 **Hardcore Mode**
   Full system audit — all pages, all components, all tokens at once.
   Best for: Quarterly reviews, initial audits, pre-launch system checks.
   Time: 5-15 minutes (depending on system size)

🎯 **Soft Mode**
   Single component deep-dive with standard shadcn/ui checks.
   Best for: Fixing one component, validating new components, debugging edge cases.
   Time: 1-3 minutes

📋 **Spec Mode**
   Single component audit against YOUR custom requirements.
   Best for: "I need this Button to support X", build-to-spec validation, custom governance.
   Time: 1-3 minutes

Which mode? (hardcore / soft / spec)
```

---

## Mode 1: Hardcore Mode (EXISTING)

Current Phases 1-5 unchanged:
1. Audit (4 layers: Technical, UX Quality, Accessibility, Design POV)
2. Dual Report (Human + Machine JSON)
3. Execution (after approval)
4. Export (tokens + components + stories + types)
5. Living Docs (HANDOFF.md)

---

## Mode 2: Soft Mode (NEW — Component Deep-Dive)

**Input:** Component page URL or component set node ID

**Workflow:**

### Phase 1S — Component Audit

Run the **8-Point Deep-Dive** on the specified component:

1. **Variant Matrix Completeness**
   - Are all combinations present? (If Size(3) × Variant(5), do you have all 15?)
   - Flag missing combos with exact names

2. **State Coverage per Variant**
   - Does every variant have: Default, Hover, Focus, Disabled?
   - Context-dependent: Error, Loading, Empty, Success, Indeterminate

3. **Token Binding Audit (per-variant)**
   - Which properties are hardcoded vs. token-bound?
   - Scan: fills, strokes, corner radius, padding, text styles, effects
   - Report: "12/15 variants use hardcoded #3B5BDB instead of --color-primary"

4. **Accessibility (per-variant)**
   - Touch target size (44×44px minimum)
   - Contrast ratio on all text/icon foregrounds
   - Focus ring present and visible on Focus state
   - Disabled state has visual cues

5. **Visual Consistency**
   - Hover has pointer cursor
   - Disabled has reduced opacity
   - Focus has visible ring
   - Transitions use motion tokens

6. **Component Documentation**
   - Description present in Figma?
   - Variant properties well-named?
   - Do/Don't examples nearby?

7. **Code Connect Status**
   - Is this component mapped to code?
   - If yes, show mapping. If no, suggest candidates.

8. **Design POV Alignment**
   - Does this component match the system's aesthetic language?
   - Radius usage consistent?
   - Motion timing consistent?

### Phase 2S — Focused Report

Output:
```markdown
# [ComponentName] Audit Report

## Variant Matrix: ✅ 15/15 complete

## State Coverage
✅ Default: All variants
✅ Hover: All variants
⚠️  Focus: Missing in 3 variants (Size=sm × Variant=ghost/outline/link)
✅ Disabled: All variants
❌ Loading: Not found (0/15)

## Token Binding: 12/15 (80%)
❌ Hardcoded values found:
- Variant=ghost: fill #F1F5F9 (should use --color-secondary)
- Variant=outline: border #CBD5E1 (should use --color-border)
- Variant=link: underline-offset 4px (should use --spacing-1)

## Accessibility
✅ Touch targets: 15/15 meet 44×44px minimum
⚠️  Contrast: Variant=ghost/disabled fails AA (2.8:1, needs 3:1)
✅ Focus rings: All variants have visible rings

## Visual Consistency
✅ Hover: pointer cursor set on all interactive variants
✅ Transitions: All use --motion-duration-200
⚠️  Disabled opacity: inconsistent (0.4 on some, 0.5 on others)

## Documentation
⚠️  Component description empty
✅ Variant properties well-named

## Code Connect
❌ Not mapped to code

## Design POV
✅ Radius matches system (--radius-lg consistently applied)
⚠️  Disabled state uses arbitrary gray instead of system muted colors

---

**Component Readiness: 72%**

Fix 6 issues before marking production-ready.
```

### Phase 3S — Component Fixes (after approval)

Apply fixes to this component only.

### Phase 4S — Component Export

Generate for this component only:
- `[Component].types.ts` — Props interface
- `[Component].stories.tsx` — Storybook stories (all variants × states)
- `[Component]-audit.md` — This report saved to disk

---

## Mode 3: Spec Mode (NEW — Custom Requirements)

**Input:** Component page URL or node ID + Custom Specifications

**Workflow:**

### Phase 1C — Capture Custom Requirements

Ask:
```
What are your requirements for this component?

You can specify:
- Missing states: "Add loading, empty, and error states"
- Size requirements: "All variants must be at least 44px tall"
- Variant additions: "I need icon-left, icon-right, and icon-only variants"
- Token requirements: "All colors must use semantic tokens, no primitives"
- Dark mode: "Must work in light and dark mode"
- Accessibility: "Focus ring must be 2px, not 1px"
- Behavior: "Disabled state should show a tooltip explaining why"
- Custom properties: "Add a 'loading' boolean prop"
- Animation: "Hover should scale to 1.02 with ease-out"

Enter your specs (one per line, or type 'done' when finished):
```

Collect all specs, then proceed.

### Phase 2C — Validate Against Specs

Run the standard 8-point audit **PLUS** validate each custom spec.

Output:
```markdown
# [ComponentName] — Custom Specification Audit

## Custom Requirements

✅ **Loading state present across all size variants**
   Found in 4/6 size variants
   ❌ Missing: Size=lg Loading, Size=icon Loading

❌ **Icon-only variant exists for Size=sm and Size=default**
   Not found
   → Need to create: Button/icon-only/sm, Button/icon-only/default

⚠️  **Touch targets ≥ 44px on all variants**
   142/156 variants meet minimum
   ❌ Below minimum: Size=sm variants (36px)
   → Recommend: Increase padding or defer with rationale

✅ **Hover states have pointer cursor**
   All 26 hover variants have pointer cursor set

⚠️  **All colors use semantic tokens**
   89% token-bound
   ❌ 17 variants have hardcoded colors

## Standard Checks
(Same 8-point audit as Soft Mode)

---

**Spec Compliance: 60%**
**Component Readiness: 78%**

Fix 3 custom requirements + 2 standard issues before marking complete.
```

### Phase 3C — Spec-Driven Fixes

Apply fixes to meet custom specs + standard issues.

### Phase 4C — Spec Report + Export

Generate:
- `[Component].types.ts` with custom props if needed
- `[Component].stories.tsx` with all required states
- `[Component]-spec-report.md` — Full validation report

---

## Updated README — Mode Comparison Table

| Feature | Hardcore | Soft | Spec |
|---|---|---|---|
| **Scope** | Full system | Single component | Single component |
| **Checks** | Standard (breadth) | Standard (depth) | Your requirements + standard |
| **Best for** | System-wide health | Component quality | Build-to-spec validation |
| **Output** | Full system report + exports | Component report + exports | Spec validation report |
| **Time** | 5-15 min | 1-3 min | 1-3 min |
| **Custom requirements** | ❌ | ❌ | ✅ |

---

## Implementation Checklist

- [ ] Update SKILL.md frontmatter description to mention 3 modes
- [ ] Add Phase 0 — Mode Selection before current Phase 1
- [ ] Wrap current Phases 1-5 under "MODE 1: HARDCORE MODE"
- [ ] Add "MODE 2: SOFT MODE" section with Phases 1S-4S
- [ ] Add "MODE 3: SPEC MODE" section with Phases 1C-4C
- [ ] Update "Start" section to trigger Phase 0 instead of going straight to audit
- [ ] Update README.md with mode comparison table
- [ ] Add mode-specific examples to README
- [ ] Remove all Apollo-specific examples, use generic shadcn examples
- [ ] Test all 3 modes on a real design system
- [ ] Tag v2.0.0

---

## Future Enhancements (Tier 1 from earlier discussion)

1. **Educational Layer** — "Why it matters" explanations for every issue
2. **Component Scaffolding** — Generate new components from scratch
3. **Pre-Handoff QA Mode** — "Is this ready for dev?" checklist
4. **System Health Tracking** — Metrics over time, trendlines

These can be added incrementally in v2.1+.
