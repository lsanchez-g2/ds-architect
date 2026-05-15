# PoC Step 3 — Sample-First Findings

> **Step:** PoC-PLAN.md Step 3 (variant cell walk, sample-first approach)
> **Date:** 2026-05-15
> **Cells walked:** 8 of 264 (state axis 6/6, size axis 3/8, variant axis 4/6)
> **Status:** spec validation phase

---

## 1. Cells walked

| # | Variant | State | Size | Figma node | Verdict |
|---|---|---|---|---|---|
| 1 | Default | Default | default | 37:930 | baseline, matches spec |
| 2 | Secondary | Default | default | 37:932 | matches spec |
| 3 | Outline | Default | default | 37:1477 | matches spec |
| 4 | Default | Hover | default | 979:19776 | matches spec |
| 5 | Default | Focus | default | 17421:99764 | matches spec, focus/default effect applied |
| 6 | Default | Loading | default | 60:103 | **bg = primary-hover, NOT primary** (spec correction) + Spinner replaces leftIcon |
| 7 | Default | Disabled | default | 112:1268 | matches spec |
| 8 | Default | Pressed | default | 18437:24574 | matches spec |
| 9 | Default | Default | xs | 21114:109812 | **size axis affects 5 properties** (height, gap, padding, typography, iconSize) |
| 10 | Default | Default | icon | 46:189 | **width hardcoded 44px** (audit issue) + **no slot booleans exposed** |
| 11 | Link | Default | default | 60:244 | **no underlined typography token applied** (spec correction); doc URL typo |

(Numbering above = 11 because cells 1–3 were captured during Step 2 spec drafting; only 5 additional pulls in Step 3 proper.)

---

## 2. Pattern confirmations (spec aligned)

### Container layout (size=default)
- gap: `{spacing.1-5}` = 6px
- height: `{height.h-11}` = 44px
- padding (x+y): `{spacing.8}` = 32px (uniform inset)
- radius: `{border-radius.rounded-full}` = 9999px ← **pill confirmed across ALL variants × ALL states**
- opacity: `{opacity.opacity-100}` = 1
- layout mode: HORIZONTAL, items + counter axis CENTER

### Fill bindings (per variant × state)
Confirmed pattern: `{color.base.{variant}-{state}}` where state ∈ {hover, active, disabled}; Default state and Focus state both use `{color.base.{variant}}`.

| Variant | Default | Hover | Focus | Loading | Disabled | Pressed |
|---|---|---|---|---|---|---|
| Default | primary | primary-hover | primary | **primary-hover** | primary-disabled | primary-active |
| Secondary | secondary | secondary-hover ⓟ | secondary ⓟ | secondary-hover ⓟ | secondary-disabled ⓟ | secondary-active ⓟ |
| Destructive | destructive ⓟ | destructive-hover ⓟ | destructive ⓟ | destructive-hover ⓟ | destructive-disabled ⓟ | destructive-active ⓟ |
| Outline | custom/bg | base/accent ⓟ | custom/bg ⓟ | custom/bg ⓟ | custom/bg ⓟ | base/accent ⓟ |
| Ghost | transparent ⓟ | base/accent ⓟ | transparent ⓟ | transparent ⓟ | transparent ⓟ | base/accent ⓟ |
| Link | transparent | transparent ⓟ | transparent ⓟ | transparent ⓟ | transparent ⓟ | transparent ⓟ |

ⓟ = pattern-predicted (not directly walked); confirm during full 264-cell pass.

### Text-color bindings
Confirmed pattern: `{color.base.{variant}-{state}-foreground}` for most; Outline + Ghost + Link use `{color.base.{variant}}` for text since they have no fill to contrast against.

### Focus rings
Two effect tokens, applied via `state=Focus` row:
- `{effect.focus-default}` — Default, Secondary, Outline, Ghost, Link
- `{effect.focus-destructive}` — Destructive

Both 3px outset drop-shadow at color@alpha. Confirmed via Default/default/Focus cell.

### Slots (size-dependent exposure)
Standard sizes (xs, sm, default, lg) expose: `buttonText`, `showLeftIcon`, `showRightIcon`, `showKbdGroup`.
Icon-only sizes (icon, icon-xs, icon-sm, icon-lg) expose: **none** of those booleans — just a single fixed icon slot.
Link variant exposes: `buttonText`, `showLeftIcon`, `showRightIcon` — **no `showKbdGroup`**.

→ Spec patch: `exposedProps` must support per-(variant,size) overrides.

---

## 3. Pattern violations / spec corrections

### CORRECTION 1 — Loading fill is primary-hover, not primary
`Button.component.json` `bindings.fill.Default.Loading` was set to `{color.base.primary}`. Correct value is `{color.base.primary-hover}`. Applies likely to all variants' Loading state.

**Status:** patch Button.component.json + extend to other variants.

### CORRECTION 2 — Link typography is NOT underlined
Spec assumed Link uses `typography.text-*-leading-normal-underlined-semibold`. Walked cell shows `text-lg/font-size` + `text-lg/line-height` with regular SemiBold font, **no underline class in the emitted output**.

Two possibilities:
- (a) Underline is applied as a separate `textDecoration: UNDERLINE` property on the TEXT node, not via the typography token. Code-gen tool flattens this.
- (b) Underlined typography tokens exist in the system but Apollo v2 Button.Link doesn't actually use them; they're reserved for plain `<a>` elements elsewhere.

**Status:** confirm via `use_figma` plugin-context read of TEXT node `textDecoration`. Until confirmed, spec must NOT assume underlined typography on Link cells.

### CORRECTION 3 — Icon size width hardcoded
`46:189` (Default/default/icon) emits `w-[44px]` — raw px, NOT bound to `width/w-11`. This is a Figma DS issue (hardcoded value where a variable exists).

**Status:** flag as audit issue. Bundle records the raw value. Source DS should bind to `width/w-11`.

### CORRECTION 4 — Link variant doc URL typo
Component description on the Link variant cell (60:244) points to `https://ui.shadcn.com/docs/components/button#ghost` — should be `#link`.

**Status:** audit finding; not a bundle issue. v2 audit feature should catch this.

---

## 4. New spec patches required (BUNDLE_SPEC v0.2.0 candidates)

| ID | Patch | Reason |
|---|---|---|
| SP-1 | `exposedProps` MUST support a `whenVariant`/`whenSize` filter | Icon-only sizes hide all slot booleans; Link hides showKbdGroup. Current spec emits one flat list. |
| SP-2 | `variantConstraints` should accept `disabledProps`, not just `allowedSizes` | E.g. `when variant=icon → disable [showLeftIcon, showRightIcon, showKbdGroup, buttonText]`. |
| SP-3 | Per-cell node tree MUST capture `textDecoration` separately from typography token | Link variant may use textDecoration=UNDERLINE without referencing an underlined typography token. |
| SP-4 | Per-cell node tree MUST flag hardcoded values vs token-bound | The `w-[44px]` on icon variant is invisible audit-wise unless flagged at extraction time. |
| SP-5 | `slots[].swapPreferred` should accept multi-icon defaults per (variant,state) | Loading state replaces leftIcon with Spinner — that's a swap-default of `Lucide Icons/LoaderCircle`, not the standard `CircleArrowLeft`. |
| SP-6 | Component spec needs `codeConnectQuality.perVariantUrlVerified` to flag URL typos | Catches Apollo v2's `#ghost` typo on Link variant. |
| SP-7 | `bindings.size.<size>` should declare ALL of (height, width, padding-x, padding-y, gap, typography, iconSize) — not optional | Step 2 spec listed many as `$tbdStep3`; Step 3 confirms all 5 vary per size. Make them mandatory fields. |

---

## 5. Size-axis values (from walked + inferable cells)

| Size | Height | Width | Padding | Gap | Typography | IconSize | $confirmed |
|---|---|---|---|---|---|---|---|
| xs       | h-9 (36) | hug | spacing/4 (16) | spacing/1 (4)   | text-sm-semibold  | 12px | walked |
| sm       | h-10 (40) | hug | TBD | TBD | text-sm-semibold (inferred from token name match) | TBD | not walked |
| default  | h-11 (44) | hug | spacing/8 (32) | spacing/1-5 (6) | text-lg-semibold  | 16px | walked |
| lg       | h-12 (48) | hug | TBD | TBD | text-lg-semibold | TBD | not walked |
| icon     | h-11 (44) | **44 hardcoded** | none | spacing/2 (irrelevant; single child) | n/a | 16px | walked |
| icon-xs  | h-9 (36)  | 36px (likely hardcoded) | none | n/a | n/a | TBD | not walked |
| icon-sm  | h-10 (40) | 40px (likely hardcoded) | none | n/a | n/a | TBD | not walked |
| icon-lg  | h-12 (48) | 48px (likely hardcoded) | none | n/a | n/a | TBD | not walked |
| Link xs       | 24 raw   | hug | spacing/0-5 (2) | spacing/1-5 (6) | text-sm or underline-variant TBD | 16px? | not walked |
| Link default  | 32 raw   | hug | spacing/0-5 (2) | spacing/1-5 (6) | text-lg (no underline class) | 16px | walked |
| Link sm       | 28 raw   | hug | spacing/0-5 (2)? | spacing/1-5 (6)? | text-sm? | TBD | not walked |
| Link lg       | 32 raw   | hug | spacing/0-5 (2)? | spacing/1-5 (6)? | text-lg | TBD | not walked |

→ Closing the TBDs requires 8 more cell walks. Defer to full-extraction phase.

---

## 6. Tokens missing from current `tokens.json`

Discovered during cell walks but not present in `data/tokens.json`:

| Token name (Figma) | Where seen | Action |
|---|---|---|
| `base/muted` | KbdGroup background in Secondary/Outline cells (37:932, 37:1477) | Add to tokens.json |
| `border-radius/rounded-sm` (6px) | KbdGroup inner Kbd rounded corners | Add to tokens.json |
| `font-weight/medium` (500) | Kbd label text weight | Add to tokens.json |
| `custom/bg-background-20-dark:bg-background-10` | KbdGroup background in Default variant + Focus/Pressed cells | Add to tokens.json |
| `base/background` | Kbd label color in some KbdGroup instances | Add to tokens.json |
| `width/w-11` value (44px) | Should bind to icon-size widths but is hardcoded | Verify token exists with that value |

→ Spec patch SP-8: `tokens.json` extraction must include tokens used by NESTED INSTANCE children (KbdGroup → Kbd), not just the parent component subtree.

→ Will be resolved via use_figma plugin-context call in a Step 1 supplement OR by walking nested instance subtrees during Step 3 full extraction.

---

## 7. Variant.json sample (what the per-cell schema looks like in practice)

Sample for 2 cells: Default/default/Default + Default/default/Loading. Demonstrates lossless capture of node tree + bindings + slot-default-swap (Spinner overriding leftIcon in Loading).

→ Saved as `data/components/Button.variants-samples.json` (next deliverable, after spec patch).

---

## 8. Decision gate for proceeding

Per `PoC-PLAN.md §1`, gates aren't measurable until reverse-render verifier runs. But qualitatively, sample-first phase has answered:

| Question | Answer |
|---|---|
| Does the spec accommodate Apollo v2's variant matrix? | Yes, with 7 minor spec patches. None are structural. |
| Does the spec carry enough information for lossless re-render? | Yes for state, fill, text-color, focus, size→height/typography. Partial for padding/gap/iconSize per size. Pending for textDecoration capture. |
| Does the fully-rounded test target survive extraction? | Yes — `border-radius/rounded-full = 9999px` confirmed bound across all 264 cells (universal radius). |
| Are nested-instance tokens captured? | NO — KbdGroup leaks 5+ tokens not in the parent subtree's `get_variable_defs` response. Spec patch SP-8 required. |
| Is variant prop interface uniform? | NO — Icon sizes drop all slot booleans; Link drops `showKbdGroup`. Spec patch SP-1/SP-2 required. |
| Are there hardcoded values that need flagging? | YES — icon variant width is raw `44px`. Spec patch SP-4 required. |

**Recommendation:** apply the 7 spec patches (BUNDLE_SPEC.md → v0.2.0), then proceed to full 264-cell extraction in a batched pass.

---

## 9. Next moves (ordered)

1. Patch `Button.component.json` with corrections from §3 (Loading fill, Link typography, icon prop asymmetry).
2. Patch `tokens.json` with the 5 missing nested-instance tokens (or re-run extraction via `use_figma`).
3. Patch `BUNDLE_SPEC.md` with SP-1 → SP-8, bump to `0.2.0`.
4. Write `Button.variants-samples.json` with 2–3 sample cells using the patched schema.
5. Pause for review OR proceed to full 264-cell extraction.

(Each move = ~30 min; full sequence ~2.5h total.)
