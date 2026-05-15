# Patterns — Apollo v2 Button

> Cross-cell + system-level conventions surfaced during PoC extraction. Useful for renderers (Claude Design, Stitch, v0, etc.) and for human reviewers reasoning about what the bundle promises.

## 1. Pill-shape is universal

Every Button cell — every variant, every state, every size, including icon-only and Link — uses `{border-radius.rounded-full}` = `9999px`. There is no `shape` axis. If you render a Button without `border-radius: 9999px`, you are not rendering an Apollo v2 Button.

This is the **canonical fidelity test** for the bundle.

## 2. Variant fill follows `{variant}-{state}` naming

Color tokens are named to match the variant prop directly:

- `state=Hover` → look up `{color.base.{variant}-hover}`
- `state=Pressed` → `{color.base.{variant}-active}` (note: "active" in the token, "Pressed" in the variant axis)
- `state=Disabled` → `{color.base.{variant}-disabled}` (tinted fill, not opacity reduction)
- `state=Default` AND `state=Focus` → `{color.base.{variant}}` (focus adds drop-shadow, doesn't change fill)
- `state=Loading` → `{color.base.{variant}-hover}` (busy-state cue) — **except** for Outline/Ghost/Link which keep their default fill

## 3. Foreground inversion only when there's a fill

- Default / Secondary / Destructive have solid fills → text uses `{color.base.{variant}-{state}-foreground}` (white-ish).
- Outline / Ghost / Link have transparent/translucent fills → text uses `{color.base.{variant}}` (no foreground inversion needed; text takes the role of the brand color).

## 4. Focus rings are always outset drop-shadows, never borders

Two effect tokens:

- `effect.focus-default` — 3px outset drop-shadow, `{color.custom.outline}` (neutral 50% alpha)
- `effect.focus-destructive` — 3px outset drop-shadow, `{color.custom.destructive-20-dark-40}` (red 20% alpha → 40% in dark)

Apply via `box-shadow: 0 0 0 3px <color>`. Do not approximate with a border.

## 5. Loading is a state + a structural swap

Loading is NOT just a color change. It's a slot rewrite:

- `leftIcon` slot becomes a **fixed Spinner component** (`swapDefaults['state=Loading']`)
- Spinner wraps `Lucide Icons / LoaderCircle`, renderer applies `animation: spin 1s linear infinite`
- `showLeftIcon` prop is **suppressed from the interface** (cannot be toggled in Loading cells)
- Renderer MUST emit `aria-busy="true"` + disable interaction

This is SP-5 in BUNDLE_SPEC v0.2.0.

## 6. Link variant breaks two assumptions

a) **Underline is on the TEXT node, not in the typography token.**
Apollo v2 has `text-{size}-leading-normal-underlined-semibold` typography tokens, but Button.Link doesn't use them. It uses standard `text-{size}-leading-normal-semibold` typography + applies `textDecoration: "UNDERLINE"` as a separate TEXT-node property. This is SP-3 v0.2.0.

b) **Link has no fill and no fixed height (heights are raw px, not h-* tokens).**
24 / 28 / 32 / 32 raw px for xs/sm/default/lg. Padding is `spacing.0-5` (2px) — minimal. Auto-layout sizing is hug both axes.

## 7. Icon-only sizes are slot-restricted

Sizes `icon`, `icon-xs`, `icon-sm`, `icon-lg`:

- No `buttonText` prop exposed
- No `showLeftIcon` / `showRightIcon` / `showKbdGroup` exposed
- Just a single, always-visible icon centered in a square frame
- Widths are hardcoded raw px (audit Finding 2) — should bind to `width.w-*` but currently don't

## 8. Asymmetric padding is the rule, not the exception

size=default uses `padding: 8px 32px` (per-side: top 8, right 32, bottom 8, left 32). The Figma code-gen `p-[var(--spacing/8, 32px)]` shorthand is **lossy** — it implies uniform padding when reality is asymmetric.

Renderers MUST honor the per-side fields in `data/components/Button.variants-samples.json` over any inferred shorthand. Claude Design proved this matters in the cell-level smoke test (it initially collapsed to shorthand, then self-corrected using the explicit per-side values — that self-correction validated SP-7).

## 9. clipsContent is variant-asymmetric

- Default variant: `clipsContent: true`
- Outline / Ghost / Link variants: `clipsContent: false`

Don't infer a uniform value. Always emit per-cell. (SP-7.)

## 10. KbdGroup background is variant-aware

When the optional `showKbdGroup=true` slot is shown:

- On Default/Destructive Button → KbdGroup background = `{color.custom.bg-background-20-dark-bg-background-10}` (translucent white)
- On Secondary/Outline Button → KbdGroup background = `{color.base.muted}` (#f5f5f5)
- On Ghost — TBD (Step 3 full)
- Not available on Link (variantConstraints)

## 11. `Lucide Icons` is the icon-set marker token

The token `icon-set.lucide-icons: true` is a boolean marker declaring the icon source. Renderers SHOULD use Lucide icons (lucide-static@1.16.0) for any slot that accepts an icon. Custom icons would require an `$extensions.customized: true` override flag on the icon's entry in `data/icons/_index.json` (not currently used in this PoC).

## 12. Source-DS findings are part of the bundle

The bundle preserves audit findings as `$auditFindings[]` on variant cells and as a separate `audit-findings-for-source.md` document. Renderers consuming this bundle should surface findings rather than render around them — the bundle reflects source-DS reality, and consumers downstream of a design system bundle are part of the audit feedback loop.
