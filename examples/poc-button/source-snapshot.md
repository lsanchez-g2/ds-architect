# Apollo v2 Button — Source Snapshot

> **Step:** PoC-PLAN.md Step 0 (snapshot source state)
> **Date:** 2026-05-15
> **Operator:** lsanchez-g2 + ds-architect
> **Source file:** `3401ZFUHoboOwA6GGjAEsq` (Apollo v2 (SA) - Design System)
> **Component set node:** `37:931`
> **URL:** https://www.figma.com/design/3401ZFUHoboOwA6GGjAEsq/Apollo-v2--SA----Design-System?node-id=37-931

---

## Visual overview

See `source-overview.png` (downloaded at maxDimension 2048; original 1254×3432).

---

## Variant matrix

The Button component set has three variant axes:

| Axis | Values | Count |
|---|---|---|
| **Variant** | Default, Secondary, Destructive, Outline, Ghost, Link | 6 |
| **State** | Default, Hover, Focus, Loading, Disabled, Pressed | 6 |
| **Size** | xs, default, sm, lg, icon, icon-xs, icon-sm, icon-lg | 8 (5 of the 6 variants) |
| **Size (Link only)** | xs, default, sm, lg | 4 |

### Cell count

| Variant | Sizes × States | Cells |
|---|---|---|
| Default | 8 × 6 | 48 |
| Secondary | 8 × 6 | 48 |
| Destructive | 8 × 6 | 48 |
| Outline | 8 × 6 | 48 |
| Ghost | 8 × 6 | 48 |
| Link | 4 × 6 | 24 |
| **Total** | | **264** |

> Note: Link has no icon-only sizes (`icon`, `icon-xs`, `icon-sm`, `icon-lg`). Bundle's variant key set must reflect that asymmetry rather than assert a uniform 8-size axis.

---

## Cell dimensions (snapshot)

Sampled cells from the Figma frame metadata. Width adapts to label content; height is fixed per size.

| Size | Height (px) | Width sample |
|---|---|---|
| xs | 36 | 79 (Default state) / 99 (Loading state, has spinner) |
| default | 44 | 124 (Default) / 146 (Loading) |
| sm | 40 | 102 / 122 |
| lg | 48 | 163 / 193 |
| icon | 44 | 44 (square) |
| icon-xs | 36 | 36 |
| icon-sm | 40 | 40 |
| icon-lg | 48 | 48 |
| Link xs | 24 | 51 / 71 (Loading) |
| Link default | 32 | 64 / 86 |
| Link sm | 28 | 58 / 78 |
| Link lg | 32 | 71 / 101 |

**Touch-target compliance flag:** sizes `xs` (36px), `sm` (40px), `icon-xs` (36px), and all Link sizes (24–32px) are below the 44×44 WCAG 2.2 AA target. This is a v2 audit finding, not a v3 extraction blocker — the bundle records the truth; downstream consumers + designers decide.

---

## Component set ID

The frame `37:931` named "Button" is the page-level wrapper holding all 264 component variants. Each variant cell is itself a COMPONENT (not a COMPONENT_SET in the strict sense — this DS uses individual components named with the `Variant=…, State=…, Size=…` pattern, then groups them visually).

> **Spec implication:** The extractor needs to either (a) infer the variant matrix from name parsing OR (b) check whether the parent node is a `COMPONENT_SET` with explicit variantProperties. For Apollo v2 we're in case (a). `BUNDLE_SPEC.md §6.2` needs a clarification note + handling for both cases. → File as v0.2.0 patch item.

---

## Cell ID examples (first per variant)

| Variant | Size | State | Figma node ID |
|---|---|---|---|
| Default | default | Default | `37:930` |
| Default | xs | Default | `21114:109812` |
| Default | sm | Default | `37:1616` |
| Default | lg | Default | `37:1658` |
| Default | icon | Default | `46:189` |
| Default | icon-xs | Default | `21114:109936` |
| Default | icon-sm | Default | `18672:198807` |
| Default | icon-lg | Default | `18672:202554` |
| Secondary | default | Default | `37:932` |
| Destructive | default | Default | `37:1475` |
| Outline | default | Default | `37:1477` |
| Ghost | default | Default | `37:1494` |
| Link | default | Default | `60:244` |

Full mapping captured in the raw `get_metadata` response (see git log for f4284dd predecessor; will be re-captured in Step 3 as `data/components/Button.variants.json`).

---

## Slot / icon usage

Inferred from cell width deltas (Loading cells are ~20px wider → spinner inserted) and the existence of `icon-*` sizes (icon-only variants). Confirmation in Step 3 via per-cell node walk.

Likely slot model:
- `iconLeft` — INSTANCE_SWAP
- `iconRight` — INSTANCE_SWAP
- `showLeftIcon` — BOOLEAN (or visibility-driven via "Has icon" boolean prop)
- `showRightIcon` — BOOLEAN
- `Loading` state internally swaps the label-row for a spinner; loading is a state, not a separate slot prop.

To confirm in Step 1/3 via `get_design_context` on the `default/Default` cell.

---

## Known fidelity concerns (pre-extraction)

1. **Asymmetric variant matrix.** Link lacks 4 of the 8 sizes other variants share. Bundle MUST emit only the cells that exist — never synthesize missing combinations.
2. **Touch-target violations.** Documented above. Bundle records actual heights; downstream consumers must not silently inflate.
3. **Loading-state width drift.** Loading variants are wider than their Default twin due to spinner. The bundle must preserve per-cell geometry, not infer from "the size".
4. **Continuous corner smoothing** (`cornerSmoothing > 0`) usage: unconfirmed. If present, this is a renderer-divergence flag per `BUNDLE_SPEC.md §13.2`.
5. **Auto-layout sizing** (hug vs fill): unconfirmed. Some variants width-adapt (`default`, `xs`, `sm`, `lg`), others are fixed-square (`icon-*`). Bundle MUST emit `primaryAxisSizingMode` per cell.

---

## Expected matrix vs. extracted matrix (gate)

| Metric | Expected | Notes |
|---|---|---|
| Total variant cells | **264** | 5 variants × 8 sizes × 6 states + 1 variant × 4 sizes × 6 states |
| Screenshots (1x + 2x) | **528 files** | 264 cells × 2 DPRs |
| Component spec files | 1 (`Button.component.json`) + 1 (`Button.variants.json`) | Per `BUNDLE_SPEC.md §6` |
| Distinct token bindings (estimate) | TBD in Step 1 | Will be counted after tokens.json emit |
| Distinct icons | TBD | Inspected in Step 4 |

If the variant cell count from Step 3 extraction ≠ 264 → Block. Investigate.

---

## Next: PoC Step 1 — Tokens extraction

Pull every variable collection used anywhere in the Button component set:
- Color (light + dark modes)
- Spacing
- Border radius
- Typography
- Motion (if present)
- Border weight (if tokenized)
- Opacity / alpha
- Icon size

Emit `data/tokens.json` per `BUNDLE_SPEC.md §4`. Verify:
- Alias chains preserved (no flattened primitives).
- Mode-aware `$value` for every semantic color.
- `$description` + `$extensions` on every semantic + component token.

Gate: `tokens.json` validates against the W3C `$type` enum + spec's mandatory-field rules.
