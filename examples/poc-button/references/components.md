# Button — Quick Reference

> Human-readable summary of `data/components/Button.component.json` + `data/components/Button.variants-samples.json`. For full machine-readable schemas, load the JSON files directly.

## Identity

- **Component:** Apollo v2 Button
- **Figma component-set node:** `37:931` (in file `3401ZFUHoboOwA6GGjAEsq`)
- **Atomic level:** atom
- **Description:** Primary interactive control. Pill-shaped (system-wide `rounded-full = 9999px`). Composes with Lucide Icon (left/right slots) and an optional Kbd shortcut group. Maps to shadcn/ui Button.

## Variant matrix — 264 cells

3 axes:

- **variant** (6): `Default` · `Secondary` · `Destructive` · `Outline` · `Ghost` · `Link`
- **state** (6): `Default` · `Hover` · `Focus` · `Loading` · `Disabled` · `Pressed`
- **size** (8 for 5 variants; 4 for Link): `xs` · `default` · `sm` · `lg` · `icon` · `icon-xs` · `icon-sm` · `icon-lg`. Link variant has only `xs/default/sm/lg`.

Total cells = 5 × 6 × 8 + 1 × 6 × 4 = **264**.

## Exposed props (with per-variant/size filters)

| Prop | Type | Default | When suppressed |
|---|---|---|---|
| `buttonText` | TEXT | "Button" | size ∈ {icon, icon-xs, icon-sm, icon-lg} |
| `showLeftIcon` | BOOLEAN | false | size ∈ {icon*}; **also state=Loading (replaced by Spinner)** |
| `showRightIcon` | BOOLEAN | false | size ∈ {icon*} |
| `showKbdGroup` | BOOLEAN | false | variant=Link; size ∈ {icon*} |

## Slots

- **leftIcon** — default: `Lucide Icons/CircleArrowLeft`. SwapDefault: when `state=Loading`, replaced by `Spinner` (local component wrapping `Lucide Icons/LoaderCircle`, animated `spin`).
- **label** — TEXT slot. Typography per-size; `textDecoration: "UNDERLINE"` on Link variant cells.
- **rightIcon** — default: `Lucide Icons/ArrowRight`.
- **kbdGroup** — nested INSTANCE component. Background variant-aware (Default → translucent `bg-background-20-dark-bg-background-10`; Secondary/Outline → `base/muted`).

## Bindings — visual identity per variant × state

### Fill

| Variant | Default | Hover | Focus | Loading | Disabled | Pressed |
|---|---|---|---|---|---|---|
| Default | primary | primary-hover | primary | **primary-hover** | primary-disabled | primary-active |
| Secondary | secondary | secondary-hover | secondary | **secondary-hover** | secondary-disabled | secondary-active |
| Destructive | destructive | destructive-hover | destructive | **destructive-hover** | destructive-disabled | destructive-active |
| Outline | custom/bg | accent | custom/bg | custom/bg | custom/bg | accent |
| Ghost | transparent | accent | transparent | transparent | transparent | accent |
| Link | transparent | transparent | transparent | transparent | transparent | transparent |

Loading uses the HOVER fill (visual busy cue). Outline/Ghost/Link Loading keeps default fill.

### Text-color

`{color.base.{variant}-{state}-foreground}` for Default/Secondary/Destructive. For Outline/Ghost/Link, text uses `{color.base.{variant}}` (no foreground inversion because there's no contrasting fill).

### Focus-ring

- Destructive → `{effect.focus-destructive}` (tinted red 20% drop-shadow)
- All others → `{effect.focus-default}` (neutral gray 50% drop-shadow)

### Radius

`{border-radius.rounded-full}` = 9999px — **universal across all 264 cells**.

### Opacity

`{opacity.opacity-100}` = 1 across all cells. Disabled state is communicated via tinted fill, not opacity reduction.

## Bindings — sizing per size

| Size | Height | Width | Padding | Gap | Typography | Icon size |
|---|---|---|---|---|---|---|
| xs | `h-9` 36 | hug | `spacing.4` 16 | `spacing.1` 4 | text-sm-semibold | 12 (raw) |
| default | `h-11` 44 | hug | **px 32 / py 8** | `spacing.1-5` 6 | text-lg-semibold | `h-4` 16 |
| sm | `h-10` 40 | hug | TBD (Step 3 full) | TBD | text-sm-semibold | TBD |
| lg | `h-12` 48 | hug | TBD | TBD | text-lg-semibold | TBD |
| icon | `h-11` 44 | **44 raw** (audit GAP-2) | 0 | n/a | n/a | `h-4` 16 |
| icon-xs | `h-9` 36 | 36 raw | 0 | n/a | n/a | 12 raw |
| icon-sm | `h-10` 40 | 40 raw | 0 | n/a | n/a | `h-4` 16 |
| icon-lg | `h-12` 48 | `h-12` 48 | 0 | n/a | n/a | `h-4` 16 |
| Link xs | 24 raw | hug | `spacing.0-5` 2 | `spacing.1-5` 6 | text-sm-semibold + UNDERLINE | `h-4` |
| Link default | 32 raw | hug | `spacing.0-5` 2 | `spacing.1-5` 6 | text-lg-semibold + UNDERLINE | `h-4` |
| Link sm | 28 raw | hug | `spacing.0-5` 2 | `spacing.1-5` 6 | text-sm-semibold + UNDERLINE | `h-4` |
| Link lg | 32 raw | hug | `spacing.0-5` 2 | `spacing.1-5` 6 | text-lg-semibold + UNDERLINE | `h-4` |

**Asymmetric padding alert:** size=default is `padding: 8px 32px`, NOT `padding: 32px`. Tailwind `p-` shorthand from Figma codegen is lossy. Always trust the per-side fields.

## Accessibility

- `role: "button"` (renderer SHOULD use a `<button>` element).
- Min touch target: 44×44 (compliant only on `default`, `lg`, `icon`, `icon-lg` — others fail WCAG 2.5.5).
- States required: Default, Hover, Focus, Loading, Disabled, Pressed.
- Focus ring per binding above.
- Loading state contract: `aria-busy="true"` + disable interaction (the consumer MUST emit this; spec promises it).

## Code Connect (shadcn parity)

Every variant maps to a shadcn/ui Button variant:

- Default → primary
- Secondary → secondary
- Destructive → destructive
- Outline → outline
- Ghost → ghost
- Link → link (**but Figma documentationLink points to `#ghost` — see Finding 3**)

## Composition

Button uses: `Lucide Icon` (left/right slots) + `Spinner` (Loading state) + `KbdGroup` (optional).

## Variant-cell samples (3 / 264)

Live in `data/components/Button.variants-samples.json`:

1. `variant=Default,state=Default,size=default` — baseline (37:930)
2. `variant=Default,state=Loading,size=default` — SP-5 swapDefaults (60:103)
3. `variant=Link,state=Default,size=default` — SP-3 textDecoration (60:244)

For any other cell, derive from the bindings tables above per the patterns validated in Step 3 sample walk + Claude Design 17/17 cell-level smoke test.

## Spec patches in play

- SP-3 — Link textDecoration captured on TEXT node, not via typography token.
- SP-4 — `$bindingStatus` per bindable property.
- SP-5 — Loading state replaces leftIcon with Spinner (`swapDefaults['state=Loading']`).
- SP-7 — Per-side padding + clipsContent always emitted.
- SP-8 — KbdGroup nested-instance tokens included in tokens.json.

See `../BUNDLE_SPEC.md §17` for full v0.2.0 changelog.

## Source-DS audit findings (not bundle bugs)

1. HIGH — Button.Link unusable inline. Typography hard-bound at 18px/semibold breaks 16px/regular body text. Recommend Apollo v2 add InlineLink component.
2. MEDIUM — Icon-size widths hardcoded (`w-[44px]` etc.) instead of bound to `width.w-*` tokens.
3. LOW — Link variant documentationLink points to shadcn `#ghost` instead of `#link`.

Full memo: `../audit-findings-for-source.md`.
