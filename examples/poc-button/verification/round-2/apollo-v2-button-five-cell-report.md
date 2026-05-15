# Apollo v2 Button — Five-Cell Render Analysis

**Scope.** Exhaustive read of five Apollo v2 Button cell renders printed from `claude.ai/design/p/019e2d3e-4f32-784e-8c03-11cece6922e8` on 2026-05-15, resolved against `Button.component.json` and `tokens.json` (mode: light). The five cells were chosen — by accident or by design — to surface every known audit signal in the bundle: SP-3, SP-4, SP-5, SP-7, GAP-1, GAP-5, GAP-7, and the WCAG 2.5.5 touch-target shortfall. This report walks each cell, then pulls cross-cell patterns into a single matrix view.

**Cell roster.**

| # | File | Variant | State | Size | Role in the bundle |
|---|------|---------|-------|------|---|
| 1 | `prompt-a.pdf` | Secondary | Hover | lg | Derived (not sampled) — lookup-order step 2 |
| 2 | `prompt-b.pdf` | Outline | Disabled | icon | GAP-5 closure cell · icon-only constraint matrix |
| 3 | `prompt-c.pdf` | Default | Default | sm | GAP-1 typography surprise · WCAG 2.5.5 fail |
| 4 | `prompt-d.pdf` | Default | Default | icon-lg | SP-4 width-via-height-token · iconSize jump |
| 5 | `prompt-e.pdf` | Outline | Loading | default | SP-5 slot swap · SP-3 a11y contract · SP-7 asymmetric padding |

---

## 1 · Per-cell readings

### 1.1 — Secondary / Hover / lg (prompt-a)

A clean derived cell. Not in the three-cell samples file, so the renderer walked lookup-order step 2: pull bindings from the component-level map and resolve against `tokens.json`.

The fill `{color.base.secondary-hover}` resolves to `#f7ffcc` — a pale lime — paired with text `{color.base.secondary-hover-foreground}` at `#3b2185`. That yellow-on-deep-purple contrast is the deliberate Secondary-Hover signature and contrasts well above WCAG AA on a 20/28 semibold body. Height is `{height.h-12}` = 48 px, padding `8 / 48` (`{spacing.2}` / `{spacing.12}`), gap `{spacing.1-5}` = 6 px. Slots off; the 16 px iconSize is bound but unused.

The annotated **surprise** here is typography: size=lg uses `text-xl` (20/28), not `text-lg`. The name says "lg" but the type ramp jumps a step. Logged in `$step2_known_gaps`. Border is `bindings.border.Secondary.$all` = none, so no border resolution work. Touch target 48 × 48, comfortably over WCAG 2.5.5.

Nothing in this cell needs rework — it is the model citizen of the five, demonstrating that step-2 derivation produces a complete, render-ready spec with no fabricated values.

### 1.2 — Outline / Disabled / icon (prompt-b)

This is the **GAP-5 closure cell** (closed 2026-05-15 via node `112:1310`). It answers a question the bundle had to settle: when an Outline button is disabled, do its border and text resolve to a dedicated `primary-disabled` token, or to `muted-foreground`? Answer: `{color.base.muted-foreground}` (`#737373`) for both border and icon. The fill stays at `{color.custom.background-dark-input-30}` (`#fafafa`) — i.e., the default Outline fill is preserved into the disabled state, not swapped.

Two more details deserve attention:

- **Opacity is held at `{opacity.opacity-100}` = 1 deliberately.** Apollo v2's disabled affordance is the muted color, not an alpha dim. Consumers must NOT stack a 0.5 opacity on top — that would compound an already-muted look and break contrast assumptions.
- **The disabled rendering contract is three-part:** HTML `disabled` + `aria-disabled="true"` + `pointer-events: none`. All three are required; the bundle treats this as a contract, not a recommendation.

The cell also surfaces two SP-4 audit signals (binding anomalies, see §2.2): `width` is hardcoded `44px` instead of binding to `{width.w-11}` (which exists in `tokens.json` at 44px), and `iconSize` is bound to `{height.h-4}` — a *height* token used as the source of a *width-equivalent* dimension. Both are source-DS issues; the bundle preserves them rather than papering over.

Touch target compliant at 44 × 44. variantConstraints suppress `buttonText`, `showLeftIcon`, `showRightIcon`, `showKbdGroup` — only the single fixed icon slot is rendered.

### 1.3 — Default / Default / sm (prompt-c)

The most audit-rich of the five, with **three signals in one cell**:

**(a) GAP-1 typography surprise.** Size `sm` maps to `{typography.text-base-leading-normal-semibold}` (16/24), not `text-sm` (14/20). The size name and the type assignment disagree by one step — the "small" button uses body type. Closed 2026-05-15 via cell `37:1616` and logged inline in `bindings.size.sm.$typographyNote`.

**(b) GAP-7 / SP-4 iconSize hardcode at 14 px.** The Apollo v2 token scale jumps from `w-3` = 12 px straight to `w-4` = 16 px — there is no `w-3-5` at 14 px. Two source-DS fixes are on the table (add the token, or rebind to an existing one); the bundle just preserves the raw `14px` as captured. Tracked in `audit-findings-for-source.md` Finding 4.

**(c) WCAG 2.5.5 touch-target failure.** 40 × 40 px ships below the 44 × 44 floor. Per `a11y.touchTargetCompliance.nonCompliant`, size `sm` joins `xs`, `icon-xs`, `icon-sm`, and all Link-* sizes as non-compliant. Practical guidance: do not use `sm` (or any of those siblings) on primary touch surfaces — escalate to `default` or larger.

Beyond the audit ledger, the cell is otherwise straightforward Default styling: fill `{color.base.primary}` `#5a35c0`, foreground `#ffffff`, no border, `clipsContent: true` (overflow: hidden), padding `8 / 24` (`{spacing.2}` / `{spacing.6}`), gap 4 px (`{spacing.1}`).

### 1.4 — Default / Default / icon-lg (prompt-d)

An icon-only cell at the largest icon size, 48 × 48. The headline finding is **SP-4 width-via-height-token**: `width` is bound to `{height.h-12}` — *a height token, used as the width source*. Other icon-* sizes hardcode width as a raw pixel value; icon-lg alone reaches into the height scale to derive its width. The semantically correct fix would be a sibling `width.w-12` token (not present in current `tokens.json`); logged in `bindings.size.icon-lg.$widthNote`.

The cell also surfaces a **surprise iconSize ramp** across the four icon-* sizes that the matrix obscures unless you tabulate them:

| Size | Container | Icon size | Δ icon | Δ container |
|---|---|---|---|---|
| icon-xs | 36 × 36 | 12 px | — | — |
| icon-sm | 40 × 40 | 16 px | +33 % | +11 % |
| icon | 44 × 44 | 16 px | 0 % | +10 % |
| icon-lg | 48 × 48 | **24 px** | **+50 %** | +9 % |

`icon-sm` and `icon` share an icon size despite different containers; `icon-lg` jumps the icon +50 % while the container grows just +9 %. The ramp is not monotonic in either dimension's growth rate. Logged in `$step2_known_gaps.GAP-1` closure note.

Default-variant `clipsContent: true` is captured here too as `overflow: hidden`. For icon-only cells the overflow rarely matters visually, but keeping it preserves round-trip fidelity — and it is the cell where you would discover, late, that an oversized icon (or a stray badge) silently gets clipped. iconSize is also flagged `$bindingStatus: partial` (raw `24px` emitted with no node-level binding).

Touch target 48 × 48, WCAG 2.5.5 ✓.

### 1.5 — Outline / Loading / default (prompt-e)

The most contract-heavy cell. Loading state activates three load-bearing rules at once:

**SP-5 — leftIcon slot swap.** `swapDefaults['state=Loading']` replaces the leftIcon (default: Lucide `CircleArrowLeft`) with a fixed `LoaderCircle` Spinner instance. `exposedBy: null` — the `showLeftIcon` prop is **suppressed from the interface**, not just defaulted off. Consumers cannot opt out of the spinner in Loading; it is always rendered.

**SP-3 — double-prevention a11y contract.** `a11y.loadingDoublePreventionContract` requires: HTML `disabled` + `aria-busy="true"` + `pointer-events: none`. The cell adds `aria-live="polite"` on top so the busy state is announced to assistive tech. This is a contract, not a suggestion — consumer code that omits any leg is non-conformant.

**SP-7 — asymmetric padding.** `padding: 8px 32px` (`{spacing.2}` / `{spacing.8}`), captured per-side. The bundle's smoke-test self-correction on cell `37:930` proved this capture is load-bearing — Tailwind `p-` shorthand would lose the asymmetry. Renderers that collapse to a single value will silently drift from the source DS.

**GAP-2 patch** appears in this cell. The Outline + Loading state holds the default fill (`{color.custom.background-dark-input-30}` = `#fafafa`), shifts text-color to `{color.base.primary-hover}` (`#4a2ba3`), and holds border at `{color.base.primary}` (`#5a35c0`) per the GAP-5 per-state border map. Note that Outline Hover and Outline Pressed are still `$tbdStep3` — not resolved in this cell, pending later extraction.

**prefers-reduced-motion** is a **renderer-side** choice, not a bundle rule: the spec doesn't define a reduced-motion variant for the Lucide LoaderCircle, so the renderer slows the spinner to a 4 s rotation rather than freezing it — a slow activity signal instead of stopping outright. Worth flagging because it is the one place across the five cells where the rendering layer makes a decision the spec doesn't dictate.

Touch target 44 × 44, WCAG 2.5.5 ✓. `cursor: progress` instead of `pointer` reinforces the in-flight state for sighted users.

---

## 2 · Cross-cell synthesis

### 2.1 — Token resolution matrix

The five cells together exercise most of the live token surface. Resolved values:

| Property | Cell 1 (Sec/Hov/lg) | Cell 2 (Out/Dis/ic) | Cell 3 (Def/Def/sm) | Cell 4 (Def/Def/ic-lg) | Cell 5 (Out/Loa/def) |
|---|---|---|---|---|---|
| fill | `#f7ffcc` | `#fafafa` | `#5a35c0` | `#5a35c0` | `#fafafa` |
| text/icon | `#3b2185` | `#737373` | `#ffffff` | `#ffffff` | `#4a2ba3` |
| border | none | `1px #737373` | none | none | `1px #5a35c0` |
| radius | 9999 px | 9999 px | 9999 px | 9999 px | 9999 px |
| height | 48 px | 44 px | 40 px | 48 px | 44 px |
| pad-x | 48 px | 0 | 24 px | 0 | 32 px |
| pad-y | 8 px | 0 | 8 px | 0 | 8 px |
| gap | 6 px | — | 4 px | — | 6 px |
| typography | xl 20/28 | — | base 16/24 | — | lg 18/28 |
| iconSize | 16 px | 16 px | 14 px | 24 px | 16 px |
| opacity | 1 | 1 | 1 | 1 | 1 |
| touch target | ✓ 48² | ✓ 44² | ✗ 40² | ✓ 48² | ✓ 44² |

The pill radius of 9999 px holds in every cell — rule 4 from the skill confirmed empirically across all five. Opacity 1 also holds in every cell, including the disabled cell (Apollo v2's disabled signal is muted color, not alpha).

### 2.2 — Binding-fidelity ledger

Token bindings in the bundle fall into four categories. The five cells together produce evidence for each:

| Category | What it means | Examples seen |
|---|---|---|
| **Fully bound** | Property resolves through `{token}` to a leaf value | All fills, foregrounds, heights, paddings (when not zero), gaps, typographies, opacities, radii |
| **Partial (`$bindingStatus: partial`)** | Token exists in resolution but the node emits a raw value with no node-level binding | iconSize on cells 2, 4 (and cell 5's spinner sizing via `{height.h-4}` used for a width-equivalent) |
| **Hardcoded** | No token reference at all; raw pixel emitted | `iconSize: 14px` on cell 3 (no `w-3-5` exists); `width: 44px` on cell 2 (despite `width.w-11` existing) |
| **Semantically mismatched** | Token bound, but the wrong axis | Cell 4 `width` → `{height.h-12}`; cell 2 / cell 5 `iconSize` → `{height.h-4}` instead of `{width.w-4}` |

The fourth category is the most interesting because it indicates the source DS lacks parallel `width.w-*` tokens at every step of the height scale — height tokens are pressed into width duty. The bundle preserves this faithfully; closing it requires upstream work on `tokens.json`.

### 2.3 — Audit-signal coverage across the five cells

| Signal | Description | Cells touching it |
|---|---|---|
| **SP-3** | Loading double-prevention a11y contract | 5 |
| **SP-4** | Width / iconSize binding anomalies | 2, 3, 4, (5 partial) |
| **SP-5** | Loading leftIcon → Spinner swap, prop suppressed | 5 |
| **SP-7** | Asymmetric padding / `clipsContent: true` on Default | 3, 4, 5 |
| **GAP-1** | size=sm typography surprise (16/24 not 14/20); also iconSize ramp note | 1 (lg→xl surprise), 3, 4 |
| **GAP-2** | Outline Loading fill/text patch (v0.2.0) | 5 |
| **GAP-5** | Outline Disabled border → `muted-foreground` (not `primary-disabled`) | 2 |
| **GAP-7** | Missing 14 px width token in scale | 3 |
| **WCAG 2.5.5** | Touch-target compliance | 1 ✓, 2 ✓, 3 ✗, 4 ✓, 5 ✓ |

Cells 3 and 5 carry the heaviest audit loads — three signals each. Cell 1 is the lightest (one surprise) and demonstrates that the step-2 derivation path produces clean, auditable output on a typical cell.

### 2.4 — Universal invariants confirmed

Across all five cells:

1. **Pill radius (`{border-radius.rounded-full}` = 9999 px) holds.** Every variant, every state, every size. Skill rule 4 verified.
2. **Open Sans 600 (semibold)** is the typography weight wherever a label is rendered. Only the size ramp varies (base / lg / xl observed; sm-the-size uses base-the-type per GAP-1).
3. **Opacity 1** holds even in Disabled. The disabled affordance is muted color, never alpha.
4. **Lucide stock icons** (CircleArrowLeft, LoaderCircle) stand in as representative payloads; stroke is applied via parent `currentColor`.
5. **variantConstraints for icon-* sizes** consistently disable `buttonText`, `showLeftIcon`, `showRightIcon`, `showKbdGroup`. Slot booleans are suppressed, not just defaulted.

### 2.5 — Where the renderer makes choices the spec doesn't

Only one such instance across the five cells: **cell 5's `prefers-reduced-motion` handling.** The bundle doesn't define a reduced-motion variant for the LoaderCircle, so the renderer chose to slow rotation to 4 s rather than freeze. This is documented as a renderer-side decision, not a bundle rule. Worth tracking because it is the seam where rendering layers can drift from each other while both remaining technically conformant.

---

## 3 · Findings and recommendations

### Source-DS issues (logged in `audit-findings-for-source.md`)

| Finding | Severity | Cell evidence | Proposed fix |
|---|---|---|---|
| Missing `width.w-11` binding on icon size | MEDIUM | Cell 2 | Rebind `bindings.size.icon.width` to existing `{width.w-11}` token |
| Missing `width.w-12` token for icon-lg | MEDIUM | Cell 4 | Add `width.w-12` = 48 px to `tokens.json`, rebind icon-lg width |
| Missing `width.w-3-5` token at 14 px | LOW | Cell 3 | Either add the token, or rebind iconSize to nearest existing (12 or 16 px) |
| iconSize bound to height tokens | LOW | Cells 2, 5 | Bind iconSize to width tokens of the same numeric value where they exist |
| size=sm typography mismatch | LOW (closed) | Cell 3 | Already documented inline; no action — rename or accept |
| size=lg → text-xl mismatch | LOW | Cell 1 | Documented in `$step2_known_gaps`; no action required |

### Consumer guidance derived from the cells

1. **Never use size `sm`, `xs`, `icon-xs`, `icon-sm`, or any Link-* size on a primary touch surface.** All five fail WCAG 2.5.5. Escalate to `default` or larger.
2. **Do not stack opacity on Disabled buttons.** The muted color is the affordance. Stacking alpha breaks contrast and compounds the dim.
3. **Loading buttons must emit all three a11y signals.** `disabled` + `aria-busy="true"` + `pointer-events: none`. The bundle treats partial implementations as non-conformant.
4. **Preserve per-side padding when implementing.** Tailwind `p-8` cannot represent `padding: 8px 32px`. Use `px-8 py-2` or equivalent four-value capture.
5. **Don't try to swap the spinner in Loading.** `showLeftIcon` is suppressed from the prop interface; the Spinner is a fixed instance.
6. **Pill radius is universal.** If you find yourself writing `border-radius` on an Apollo v2 Button, you are drifting from the spec.

### Bundle-side observations (not bugs)

- **261 of 264 cells still pending full node-tree serialization.** The bindings map covers the remainder predictably; the five cells under review confirm that step-2 derivation works.
- **Dark mode is not in this bundle.** Every value resolved here is light-mode. A dark-mode pass would need to re-run extraction once Figma plugin context is available.
- **Outline Hover and Outline Pressed remain `$tbdStep3`.** Cell 5 demonstrates that Outline Loading is patched in v0.2.0; the two intermediate interactive states for the same variant are not yet resolved.

---

## 4 · Conclusion

The five cells form an unusually well-chosen audit sample. Between them they exercise:

- All three lookup orders (verbatim sample, bindings-map derivation, token resolution)
- Three of six variants (Secondary, Outline, Default — Destructive, Ghost, Link not represented)
- Four of six states (Default, Hover, Disabled, Loading — Focus and Pressed not represented)
- Five of eight sizes (sm, default, lg, icon, icon-lg — xs, icon-xs, icon-sm not represented)
- Every audit signal currently logged in the bundle (SP-3, SP-4, SP-5, SP-7, GAP-1, GAP-2, GAP-5, GAP-7)
- Both pass and fail cases for WCAG 2.5.5

What is **not** covered: the Focus state's focus-ring resolution (every cell shows `focus-ring: n/a`); the Link variant's `textDecoration: UNDERLINE` rule (SP-6, none of the five cells is a Link); the Destructive variant entirely; and the smallest sizes (xs, icon-xs, icon-sm). A follow-up extraction targeting any Focus state, any Link cell, and a Destructive cell would close the remaining coverage gaps and put eyes on the bundle's last untested contracts.

The renderer itself is conformant: every property is sourced from a token or a variant cell (rule 1 honored), the pill radius is universal (rule 4 verified), the Loading contracts fire (rules 5 and 8 honored), and asymmetric padding is preserved per-side (rule 7 honored). The single renderer-side judgment call — the reduced-motion spinner slowdown — is documented in the cell rather than buried, which is the behavior the bundle's "audit findings are part of the bundle" rule (rule 10) asks for.
