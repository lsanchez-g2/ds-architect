---
name: apollo-v2-switch-bundle-poc
description: Apollo v2 Switch design-system bundle (PoC, v0.3.0). 64-cell variant matrix across 5 axes (size × active × state × type × controlPlacement), 39 tokens, no icons, full node trees for 3 sampled cells, light mode. Apply when rendering, prototyping, or generating UI from Apollo v2 Switch — Off/On toggle states × Default/Disabled/Focus/Invalid × default/sm × Default/Box × Start/End. Resolves Figma binding tokens against tokens.json; honors SP-9 motionReduce policy=skip on toggle transition (Off↔On); validates SP-9 across both motion patterns (Spinner uses slow, Switch uses skip).
---

# Apollo v2 Switch — Lossless Design-System Bundle (PoC)

Source of truth for Apollo v2 Switch rendering and prototyping. Second component PoC after Button. Validates BUNDLE_SPEC v0.3.0 on a non-Button atom with a richer variant matrix (5 axes vs Button's 3).

## What this bundle contains

- **`MANIFEST.json`** — bundle header: version, source file, counts, SHA256 checksums.
- **`data/tokens.json`** — 39 W3C-extended design tokens (single mode: light). 20 shared with Button bundle (primary, destructive, muted-foreground, font-sans, focus effects, etc.); 17 unique to Switch (w-8, h-3, border-2, rounded-lg, font-weight/normal, 3 typography compounds, spacing/2-5 + spacing/3, opacity-50, base/foreground + base/border + base/ring, 3 custom alpha colors).
- **`data/components/Switch.component.json`** — 64-cell variant matrix declaration across 5 axes, exposedProps, bindings by axis-combination (track-fill by active+state, focus-ring by state, opacity by state, etc.), slots (track, thumb, label, description), a11y, codeConnect.
- **`data/components/Switch.variants-samples.json`** — 3 of 64 cells serialized as full recursive node trees (Default/Off baseline + Default/On active flip + Default/Invalid state). Each carries `$bindingStatus`, audit signal notes, `$verificationCandidate.expectations` block.
- **`data/icons/_index.json`** — empty (Switch uses no icons; structural marker preserved).
- **`references/`** — human-readable summaries (tokens.md, components.md, patterns.md). Load these for progressive disclosure without consuming the full JSON data files.
- **`verification/`** — reverse-render artifacts (pending Step 8 Claude Design re-test).

## Lookup order

1. **`data/components/Switch.variants-samples.json`** — if the requested cell is among the 3 sampled (Default/Off, Default/On, Default/Invalid), render verbatim.
2. **`data/components/Switch.component.json`** — else derive from the component-level `bindings` map. Per-axis-combination bindings cover all 64 cells.
3. **`data/tokens.json`** — resolve every `{path.to.token}` reference. Active mode = light.
4. **`data/icons/_index.json`** — empty; no icons to resolve.

## Hard rules

1. **Never invent values.** Every property comes from a token or a variant cell. Refuse rather than fabricate.
2. **Resolve `{path.to.token}` against `data/tokens.json`.** Light mode only.
3. **Honor `boundVariable` over inline values.** Binding wins.
4. **Pill radius (`{border-radius.rounded-full}` = 9999px) is universal on track + thumb.** Box-type frame uses `{border-radius.rounded-lg}` (10px) — the outer container only, not the toggle itself.
5. **`role="switch"` + `aria-checked` are mandatory.** Off → `aria-checked="false"`; On → `aria-checked="true"`.
6. **Active-axis mechanism is layout-driven, not position-driven.** Off → track `justify-content: flex-start`; On → `flex-end`. Single-child flex.
7. **Disabled = `opacity: 0.5` on the OUTER `<button>` container.** Dims the whole component uniformly. Do NOT apply to inner elements individually.
8. **Invalid state shows the destructive focus ring AT REST.** Always visible, not only on Focus. F11 audit signal — Apollo v2 intentional emphasis.
9. **Track-border swaps per state:** Default/Disabled use `border-2 transparent` (breathing margin); Focus uses `border-1 base/ring`; Invalid uses `border-1 destructive`.
10. **SP-9 motionReduce policy = `skip` on toggle transition.** Reduced-motion users see Off→On as instant variant change (no slide).

## Known gaps + planned work

- 61 of 64 cells still pending full node-tree serialization. The bindings map in `Switch.component.json` covers the remainder predictably.
- Dark mode values not yet extracted.
- Step 5 (prototype.json) deferred — toggle animation timing (duration + easing) inferred only.
- Source-DS audit signals (F5–F11): Switch lacks Hover/Pressed/Loading states (F5/F6/F7); opacity-vs-tinted inconsistency with Button (F8); raw track heights (F9 default 18.4px, F10 sm 14px); Invalid-ring-at-rest documented (F11).
- Container widths (312/304/300/296) hardcoded raw, not bound.

## Provenance

- **Source Figma file:** `3401ZFUHoboOwA6GGjAEsq` (Apollo v2 (SA) - Design System).
- **Source component-set node:** `60:450` (Switch).
- **Extractor:** `ds-architect@v3` (https://github.com/lsanchez-g2/ds-architect).
- **Spec version:** `BUNDLE_SPEC.md@v0.3.0` (DRAFT — locks when re-test with both Button.Loading and Switch.toggle motion-reduce policies validates).
- **License (bundle):** inherits `lsanchez-g2/ds-architect` MIT license.

## Companion bundle

This bundle is a sibling to `apollo-v2-button-bundle-poc`. Both share the same source DS and most of the token vocabulary. Switch is the SECOND component to validate BUNDLE_SPEC v0.3.0 end-to-end. Together they prove the contract generalizes beyond a single component.
