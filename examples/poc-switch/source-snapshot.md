# Apollo v2 Switch — Source Snapshot

> **Step:** PoC-PLAN.md Step 0 (snapshot source state)
> **Date:** 2026-05-15 (evening)
> **Operator:** lsanchez-g2 + ds-architect@v3
> **Source file:** `3401ZFUHoboOwA6GGjAEsq` (Apollo v2 (SA) - Design System)
> **Component set node:** `60:450`
> **URL:** https://www.figma.com/design/3401ZFUHoboOwA6GGjAEsq/Apollo-v2--SA----Design-System?node-id=60-450

---

## Visual overview

See `source-overview.png` (downloaded at maxDimension 2048; original 1416×1724).

---

## Variant matrix — **5 axes, 64 cells**

Apollo v2 Switch is significantly more complex than Button along the variant-axis dimension. Button has 3 axes (variant × state × size). Switch has 5:

| Axis | Values | Count |
|---|---|---|
| **Size** | `default`, `sm` | 2 |
| **Active** | `Off`, `On` | 2 |
| **State** | `Default`, `Disabled`, `Focus`, `Invalid` | 4 |
| **Type** | `Default`, `Box` | 2 |
| **Control Placement** | `Start`, `End` | 2 |

Total cells: 2 × 2 × 4 × 2 × 2 = **64**.

---

## Cell dimensions (sampled from metadata)

Type = `Default` cells (compact, inline-label):
| Size | Width | Height | Touch target |
|---|---|---|---|
| default | 312 | 42 | ✗ (height < 44) |
| sm      | 304 | 36 | ✗ |

Type = `Box` cells (boxed/contained, labeled variant):
| Size | Width | Height | Touch target |
|---|---|---|---|
| default | 300 | 62 | ✓ |
| sm      | 296 | 56 | ✓ (≥ 44) |

→ **Half the matrix fails WCAG 2.5.5** for the touch-target dimension (all Type=Default cells across both sizes). Audit signal.

Some Active=On + Box cells emit 292w vs 300w — slight width drift, likely intentional for thumb-shift compensation. Verify in Step 3.

---

## Axis-axis observations

### State axis = 4 values, missing 2 common ones

Switch declares only `Default`, `Disabled`, `Focus`, `Invalid`. Compare to Button's state axis (`Default`, `Hover`, `Focus`, `Loading`, `Disabled`, `Pressed`).

| Missing state | Switch implication |
|---|---|
| `Hover` | No mouse-hover visual feedback on the thumb or track |
| `Pressed` | No active-press feedback — toggling has no down-state cue |
| `Loading` | No async-toggle busy state |

**Audit signal candidates:**
- Finding 5 (likely MEDIUM): Switch lacks Hover state. Inconsistent with Button which has it. Recommend adding Hover state with the same `*-hover` token pattern.
- Finding 6 (likely LOW): Switch lacks Pressed state. Toggle interaction has no visible feedback during press. Recommend adding.

Both findings are source-DS gaps. Bundle reflects what exists.

### `Invalid` state

Switch has its own `Invalid` state (red/error styling). Button uses `Destructive` variant for the same purpose. Inconsistency across the system — neither approach is wrong but it's worth flagging.

Audit signal candidate: Finding 7 (LOW) — state-system inconsistency. Either standardize on "Error" / "Invalid" / "Destructive" naming OR document why each component uses its own convention.

### `Type` = Default vs Box

Box type frames are taller (62h default, 56h sm) and presumably include a labeled container. Default type is compact inline. Inferred from dimensions; confirm in Step 3 via cell walk.

Likely: Type=Box is for form-field placements where Switch should match the surrounding input-field surface area. Type=Default is for inline lists / settings rows.

### `Control Placement` = Start vs End

Start = toggle on the LEFT of the label. End = toggle on the RIGHT. Common pattern; reasonable.

→ This is a **layout-only axis** — it doesn't change visual identity of the toggle itself, just left/right position. Bundle could optimize: capture once + emit per-placement positional bindings. Don't double-extract.

### `Active` = Off vs On

Binary boolean toggle state. Maps to Button's `variant` in semantic role (visual identity changes between Off/On). Different bindings expected per Active value.

---

## Animation expectations (motion-reduce relevance)

**Switch is the key SP-9 test target outside of Spinner.** The toggle from Off → On (and back) is an animation: the thumb slides horizontally + the track changes color. Apollo v2 source DS likely defines this via Figma prototype interactions (CHANGE_VARIANT smart-animate).

PoC pipeline must capture:
- Transition trigger (likely `ON_CLICK` → CHANGE_VARIANT toggling Active)
- Duration (likely `150–300ms` per Apollo v2 motion taste)
- Easing (likely `linear` or `ease-out`)
- **motionReduce policy** — what should reduced-motion users see?

Recommended motionReduce policy candidates for Switch:
- `skip` (the most likely correct answer): jump directly to end state. No motion, full information preserved. Toggle still works; user sees Off → On with no intermediate frames.
- `slow`: 4× duration like Spinner. **Worse** here — slowing the toggle to ~1s on every click makes the UI feel laggy without preserving meaningful information.
- `freeze`: nonsensical for a state transition (would leave the thumb stuck mid-slide).

→ **Validates SP-9 across both motion patterns**: looping animation (Spinner → `slow`) vs state-transition animation (Switch → `skip`). Closes the v0.3.0 lock loop.

---

## Cell ID examples (first per axis combination)

Representative cells for sample-first extraction:

| Axis combination | Cell ID |
|---|---|
| Size=default, Active=Off, State=Default, Type=Default, ControlPlacement=Start | `428:1425` |
| Size=default, Active=On,  State=Default, Type=Default, ControlPlacement=Start | `428:1431` |
| Size=default, Active=Off, State=Disabled, Type=Default, ControlPlacement=Start | `21141:13977` |
| Size=default, Active=Off, State=Focus, Type=Default, ControlPlacement=Start | `17428:105516` |
| Size=default, Active=Off, State=Invalid, Type=Default, ControlPlacement=Start | `21141:9448` |
| Size=default, Active=Off, State=Default, Type=Box, ControlPlacement=Start | `17119:18448` |
| Size=default, Active=Off, State=Default, Type=Default, ControlPlacement=End | `17118:18360` |
| Size=sm, Active=Off, State=Default, Type=Default, ControlPlacement=Start | `21136:10697` |
| Size=default, Active=On, State=Focus, Type=Default, ControlPlacement=Start | `17428:105544` |

Sample-first plan: walk ~8–10 cells spanning all 5 axes. Confirms binding patterns. Bulk-extract remaining 54–56 cells via Tier-2 bindings map (proven workflow from Button PoC).

---

## Pre-extraction concerns (audit signals queued)

1. **Touch-target violations** — all Type=Default cells (32 of 64) fail WCAG 2.5.5 (h=42 or h=36). Recorded; not a bundle blocker.
2. **Missing Hover state** — Switch has no Hover state across the 4-value state axis. Source-DS gap (audit Finding 5 candidate).
3. **Missing Pressed state** — Switch has no Pressed/active-press state. Source-DS gap (audit Finding 6 candidate).
4. **State naming inconsistency** — Switch uses `Invalid`; Button uses `Destructive` variant + state combo. System-level inconsistency (audit Finding 7 candidate).
5. **5-axis matrix not yet stress-tested in spec** — Button used 3 axes max; Switch's 5 axes will exercise `variantProperties.{axisName}` declaration limits + per-cell key serialization (`variant=…,state=…,size=…,type=…,placement=…`).
6. **No icons used inside Switch** — unlike Button which composes Lucide icons. Switch icons (if any) limited to error indicator on Invalid state. Confirm in Step 3.
7. **Apollo v2 likely uses Figma prototype interactions for the toggle animation** — need to capture via prototype.json (PoC Step 5 → not done for Button, would be new territory). May trigger spec patches.

---

## Stress-test value for the v0.3.0 spec

Switch will validate or break these v0.3.0 features:

| Feature | Switch test |
|---|---|
| SP-9 motionReduce on state-transition (not Loading) | Toggle animation needs `skip` policy — validates policy enum across both `slow` and `skip` |
| SP-4 `semantically-mismatched` $bindingStatus | Likely fires on track/thumb sizing if dimensions are pressed cross-axis |
| 5-axis variantProperties declaration | Schema works with any number of axes? Confirm. |
| variantConstraints across more axes | Are any (Active × State) or (Type × Size) combinations forbidden? Likely no, but spec should accommodate if yes. |
| Loading-state absence | Confirms not every Button-state pattern transfers — Switch genuinely lacks Loading. Spec should not force it. |

---

## Expected matrix vs. extracted matrix (gate)

| Metric | Expected | Notes |
|---|---|---|
| Total variant cells | **64** | 2 × 2 × 4 × 2 × 2 |
| Screenshots (1x + 2x) | **128 files** | 64 cells × 2 DPRs |
| Component spec files | 1 (`Switch.component.json`) + 1 (`Switch.variants-samples.json`) | Per `BUNDLE_SPEC.md` §6 |
| Distinct token bindings (estimate) | TBD in Step 1 | Likely overlap heavily with Button's 84 — track/thumb colors are new + spacing/radius reuse |
| Distinct icons | 0–1 | Only if Invalid state uses an error icon |
| Distinct prototype interactions | ≥ 1 | Active=Off ↔ Active=On toggle |

If Step 3 extraction cell count ≠ 64 → block. Investigate.

---

## Next: PoC Step 1 — Tokens delta

Pull every variable used in Switch subtree. Most will overlap with Button's tokens.json (already 84 leaves). Net-new tokens expected:

- Switch track color tokens (Off / On / Disabled / Focus / Invalid)
- Switch thumb color tokens (Off / On / Disabled / Focus / Invalid)
- Switch track size tokens (likely reuse height/width scale)
- Possible motion duration token for the toggle (if Apollo v2 has a `motion/duration/*` collection)

Append the delta to existing `examples/poc-button/data/tokens.json` OR emit a separate `examples/poc-switch/data/tokens.json` — depends on whether bundles share token files or each component bundle stands alone.

**Decision needed:** shared tokens.json (single source of truth across all Apollo v2 components) vs per-component tokens.json (each bundle is self-contained for ZIP distribution).

For PoC scope, emit standalone `examples/poc-switch/data/tokens.json` to keep the Button bundle untouched. After atom batch, decide on a consolidated `apollo-v2-tokens.json` shared across all component bundles.
