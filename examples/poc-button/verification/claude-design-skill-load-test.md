# Apollo v2 Button — Session Render Report

**Date:** 2026-05-15
**Skill:** `apollo-v2-button-bundle-poc` (v0.2.0 PoC, 264-cell matrix, 3 cells sampled, 261 derived from bindings map)
**Source:** Figma file `3401ZFUHoboOwA6GGjAEsq`, component-set `37:931` (Apollo v2 Button)
**Bundle path:** `/Users/gsancheztorres/.claude/skills/apollo-v2-button-bundle-poc/`

---

## Session prompts (5)

| # | Prompt | Lookup tier | Status |
|---|---|---|---|
| 1 | Render variant=Default, state=Default, size=default | Tier 1 (samples file) | Fully resolved |
| 2 | Render variant=Secondary, state=Hover, size=lg | Tier 2 (component bindings) | **Partial — `$tbdStep3` gap** |
| 3 | Render variant=Outline, state=Disabled, size=icon | Tier 2 (component bindings) | Partial — audit flags F2 + GAP-5 |
| 4 | Render variant=Destructive, state=Loading, size=default | Tier 2 (component bindings) | Fully resolved (SP-5 swap applied) |
| 5 | Create report file for last 5 prompts | Meta — this file | Done |

---

## Prompt 1 — Default / Default / default

**Tier 1 hit.** Direct serialization from `Button.variants-samples.json` (node `37:930`).

### Resolved CSS
```css
.btn {
  display: inline-flex; align-items: center; justify-content: center;
  gap: 6px;                              /* {spacing.1-5} */
  height: 44px;                          /* {height.h-11} */
  padding: 8px 32px;                     /* {spacing.2} / {spacing.8} */
  background-color: #5a35c0;             /* {color.base.primary} */
  color: #ffffff;                        /* {color.base.primary-foreground} */
  border: 0;
  border-radius: 9999px;                 /* {border-radius.rounded-full} */
  overflow: hidden;                      /* clipsContent: true (Default variant only) */
  font-family: "Open Sans", sans-serif;
  font-weight: 600;
  font-size: 18px;
  line-height: 28px;
}
```

### Token resolution
| Property | Token | Value |
|---|---|---|
| background-color | `{color.base.primary}` | `#5a35c0` |
| color | `{color.base.primary-foreground}` | `#ffffff` |
| border-radius | `{border-radius.rounded-full}` | `9999px` |
| padding-y / padding-x | `{spacing.2}` / `{spacing.8}` | `8px` / `32px` |
| gap | `{spacing.1-5}` | `6px` |
| height | `{height.h-11}` | `44px` |
| typography | `{typography.text-lg-leading-normal-semibold}` | Open Sans 600 / 18px / 28px |

### Status
- `$bindingStatus: "fully-bound"` on layout / fills / corners / text
- Slots all `false` (showLeftIcon, showRightIcon, showKbdGroup)
- Verified: `verification/claude-design-3-cell-report.html` (round-tripped 2026-05-15, 8/8 CSS predicates)

---

## Prompt 2 — Secondary / Hover / lg

**Tier 2.** Component bindings + tokens.json. **PARTIAL RENDER — `$tbdStep3` gap on padding/gap/iconSize.**

### Resolved (bound)
| Property | Token | Value |
|---|---|---|
| background-color | `{color.base.secondary-hover}` | `#f7ffcc` |
| color | `{color.base.secondary-hover-foreground}` | `#3b2185` |
| border | `bindings.border.Secondary.$all` | `none` |
| border-radius | `{border-radius.rounded-full}` | `9999px` |
| height | `{height.h-12}` | `48px` |
| typography | `{typography.text-lg-leading-normal-semibold}` | Open Sans 600 / 18px / 28px |
| overflow | clipsContent.others | `visible` |

### Unbound — refused per Hard Rule #1
```json
"$tbdStep3": ["padding-x", "padding-y", "iconSize", "gap"]
```
- `padding-x` — no value (pattern hint only: proportional scale)
- `padding-y` — no value
- `gap` — no value
- `iconSize` — no value

### Partial CSS
```css
.btn-secondary-hover-lg {
  display: inline-flex; align-items: center; justify-content: center;
  /* gap: <UNBOUND $tbdStep3> */
  height: 48px;
  /* padding: <UNBOUND $tbdStep3> */
  background-color: #f7ffcc;
  color: #3b2185;
  border: 0;
  border-radius: 9999px;
  overflow: visible;
  font-family: "Open Sans", sans-serif;
  font-weight: 600;
  font-size: 18px;
  line-height: 28px;
}
```

### Action required
Walk Apollo v2 cell `37:1658` (Secondary,Default,lg) via `use_figma`. Capture per-side padding + itemSpacing. Backfill `bindings.size.lg` in `Button.component.json`. Re-render.

---

## Prompt 3 — Outline / Disabled / icon

**Tier 2.** Multiple audit flags fire on this cell.

### Resolved
| Property | Token | Value | Status |
|---|---|---|---|
| background | `{color.custom.background-dark-input-30}` | `#fafafa` | bound |
| color (icon) | `{color.base.primary-disabled}` | `#ede9fe` | `$tbdStep3` inferred |
| border-color | `{color.base.primary}` | `#5a35c0` | `$tbdStep3` "likely shifts on Disabled" |
| border-width | `{border-width.border}` | `1px` | bound |
| border-radius | `{border-radius.rounded-full}` | `9999px` | bound |
| height | `{height.h-11}` | `44px` | bound |
| width | `bindings.size.icon.width` | `44px` | **HARDCODED — F2 audit** |
| padding | `0` | `0` | bound |
| icon size | `{height.h-4}` | `16px` | bound |
| overflow | clipsContent.others | `visible` | bound |

### Variant constraints (size=icon-*)
```
disabledProps: ["buttonText", "showLeftIcon", "showRightIcon", "showKbdGroup"]
```
Single fixed icon slot. Confirmed via cell `46:189`. Icon identity falls back to `slots.leftIcon.default` = Lucide `CircleArrowLeft`.

### Resolved CSS
```css
.btn-outline-disabled-icon {
  display: inline-flex; align-items: center; justify-content: center;
  width: 44px;                           /* HARDCODED — should bind {width.w-11} */
  height: 44px;                          /* {height.h-11} */
  padding: 0;
  background-color: #fafafa;             /* {color.custom.background-dark-input-30} */
  color: #ede9fe;                        /* {color.base.primary-disabled} — INFERRED */
  border: 1px solid #5a35c0;             /* border-color UNCONFIRMED on Disabled */
  border-radius: 9999px;
  overflow: visible;
  cursor: not-allowed;
  pointer-events: none;
}

.btn-outline-disabled-icon > svg {
  width: 16px;
  height: 16px;
  stroke: currentColor;
  stroke-width: 2;
  fill: none;
}
```

### Audit signals
1. **F2 (MEDIUM)** — width `44px` hardcoded, should bind `{width.w-11}`
2. **GAP-5 (open)** — Outline Disabled border color UNCONFIRMED, cell `112:1310` not walked
3. **text-color $tbdStep3** — Outline.Disabled color inferred, low-contrast vs background (intentional "disabled" cue, but unconfirmed)

### A11y
Disabled state → `disabled` + `aria-disabled="true"` + `pointer-events: none`. `aria-label` mandatory (icon-only, no text).

---

## Prompt 4 — Destructive / Loading / default

**Tier 2.** Loading-state swap (SP-5) applied. Fully resolved.

### Resolved
| Property | Token | Value |
|---|---|---|
| background | `{color.base.destructive-hover}` | `#b91c1c` |
| color | `{color.base.destructive-foreground}` | `#ffffff` |
| border | `bindings.border.Destructive.$all` | `none` |
| border-radius | `{border-radius.rounded-full}` | `9999px` |
| height | `{height.h-11}` | `44px` |
| padding | `{spacing.2}` / `{spacing.8}` | `8px 32px` |
| gap | `{spacing.1-5}` | `6px` |
| typography | `{typography.text-lg-leading-normal-semibold}` | Open Sans 600 / 18px / 28px |
| icon size | `{height.h-4}` | `16px` |
| overflow | clipsContent.others | `visible` |

### SP-5 swap
`slots.leftIcon.swapDefaults['state=Loading']`:
- `library: Local`, `icon: Spinner`
- `innerIcon: Lucide LoaderCircle`
- `animation: spin`
- `showLeftIcon` prop **suppressed** from variant interface

### A11y (loadingDoublePreventionContract)
```
aria-busy="true" + disabled (or pointer-events: none)
```
Non-negotiable per `Button.component.json.a11y`.

### Resolved CSS
```css
.btn-destructive-loading-default {
  display: inline-flex; align-items: center; justify-content: center;
  gap: 6px;
  height: 44px;
  padding: 8px 32px;
  background-color: #b91c1c;             /* destructive-hover (Loading uses hover fill) */
  color: #ffffff;
  border: 0;
  border-radius: 9999px;
  overflow: visible;
  cursor: not-allowed;
  pointer-events: none;
  font-family: "Open Sans", sans-serif;
  font-weight: 600;
  font-size: 18px;
  line-height: 28px;
}

.btn-destructive-loading-default > .spinner {
  width: 16px;
  height: 16px;
  stroke: currentColor;
  stroke-width: 2;
  fill: none;
  animation: spin 1s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }
```

### Pattern confirmed
Loading fill = `*-hover` token across primary/secondary/destructive (visual busy cue). Outline/Ghost/Link Loading retain their default fills.

---

## Cross-cell findings summary

### Source-DS audit signals surfaced (bundle-faithful, not bundle bugs)

| ID | Severity | Cell(s) | Issue |
|---|---|---|---|
| F1 | HIGH | Link/* | `text-lg-semibold` 18px breaks inline body-text rhythm. Apollo v2 needs `InlineLink` or `size=inline`. |
| F2 | MEDIUM | icon/icon-xs/icon-sm | width hardcoded (`44px`/`36px`/`40px`) instead of binding to width tokens |
| F3 | LOW | Link.documentationLink | Figma points to shadcn `#ghost` instead of `#link` |

### Bundle gaps (extraction TODOs)

| ID | Cells affected | Resolution |
|---|---|---|
| GAP-1 | size=sm/lg + icon-xs/icon-sm/icon-lg padding/gap/iconSize | Walk cells `37:1616`, `37:1658`, etc. |
| GAP-2 | Outline/Ghost Hover/Pressed, Link all text-color states | Verify per-cell |
| GAP-5 | Outline Disabled border color | Walk cell `112:1310` |

### Lookup tier distribution

- **Tier 1 (samples, full node tree):** 1/4 prompts (Default/Default/default)
- **Tier 2 (bindings map):** 3/4 prompts (Secondary/Hover/lg, Outline/Disabled/icon, Destructive/Loading/default)
- Tier 2 cells with `$tbdStep3` gaps: 2 (Secondary lg padding, Outline Disabled border)
- Tier 2 cells fully resolved from bindings: 1 (Destructive Loading default — size=default + state pattern both confirmed)

### Confidence ladder

| Cell | Confidence | Reason |
|---|---|---|
| Default / Default / default | HIGH | Full node-tree sample, round-trip verified |
| Destructive / Loading / default | HIGH | size=default confirmed (3 cells walked), Loading swap declarative |
| Outline / Disabled / icon | MEDIUM | size=icon walked (`46:189`) but Disabled state border/color flagged `$tbdStep3` |
| Secondary / Hover / lg | LOW | size=lg padding/gap unbound; only height + fill resolvable |

---

## Hard rules upheld

| Rule | Applied at |
|---|---|
| #1 — Never invent values | Prompt 2: refused to fabricate Secondary/lg padding despite "proportional scale" hint |
| #2 — Resolve `{path.to.token}` against tokens.json | All 4 cells |
| #3 — `boundVariable` wins over inline | Prompt 1 (sample has both; bindings honored) |
| #4 — `border-radius: 9999px` universal | All 4 cells — pill confirmed across Default/Secondary/Outline/Destructive |
| #5 — Loading mandates `aria-busy` | Prompt 4: contract emitted in HTML output |
| #6 — `textDecoration` independent of typography | n/a (no Link cell in renders 2-4) |
| #7 — Asymmetric padding per-side | Prompt 1 + Prompt 4: `8px 32px` not `32px` |
| #8 — Loading swaps leftIcon → Spinner | Prompt 4: `showLeftIcon` suppressed, spinner emitted |
| #9 — variantConstraints filter slots | Prompt 3: icon-* size → all slot booleans + `buttonText` disabled |
| #10 — Audit findings surfaced, not hidden | Prompt 3: F2 + GAP-5 named in output |

---

## Recommended next actions

1. **Walk cell `37:1658`** (Secondary,Default,lg) — closes GAP-1 for size=lg, enables full Prompt 2 re-render
2. **Walk cell `112:1310`** (Outline,Disabled,*) — closes GAP-5, confirms Prompt 3 border color
3. **Fix F3 in source DS** — repoint Link.documentationLink from shadcn `#ghost` → `#link` (low-effort, ships immediately)
4. **Backfill width tokens** (`w-9`, `w-10`, `w-11`, `w-12`) and bind icon-* widths — closes F2
