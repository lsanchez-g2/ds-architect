# Switch Tokens — Quick Reference

> Summary of `data/tokens.json` (39 leaves). 20 shared with Button bundle; 17 unique to Switch. Source: Apollo v2 file `3401ZFUHoboOwA6GGjAEsq`, Switch-subtree (node 60:450).

## Colors

### Semantic (`base/*`)

| Token | Value | Role in Switch |
|---|---|---|
| `base/primary`           | #5a35c0 | Active=On track fill |
| `base/destructive`       | #dc2626 | Invalid state — border, label, ring tint base |
| `base/background`        | #fafafa | Thumb fill (constant across all states), Box-type container fill |
| `base/foreground`        | #0a0a0a | Label text (Default/Disabled/Focus states) — **NEW vs Button** |
| `base/muted-foreground`  | #737373 | Description text (always), Disabled label tone via opacity |
| `base/border`            | #e5e5e5 | Box-type container border — **NEW vs Button** |
| `base/ring`              | #5a35c0 | Focus border — **NEW vs Button**; same hex as primary, semantically distinct |

### Custom (alpha + dark-mode-aware)

| Token | Value | Role |
|---|---|---|
| `custom/outline`                              | #a3a3a380 | focus-default ring tint |
| `custom/destructive-10-dark-20`               | #dc26261a | focus-destructive ring base alpha |
| `custom/destructive-20-dark-40`               | #dc262633 | focus-destructive ring spread alpha |
| `custom/input-dark-input-80`                  | #a3a3a3   | Track fill when Active=Off — **NEW vs Button** |
| `custom/bg-primary-5-dark-bg-primary-10`      | #1717170d | Subtle overlay (reserved) — **NEW vs Button** |
| `custom/alpha-30-dark-alpha-20`               | #ffffffb2 | White 70% alpha (reserved) — **NEW vs Button** |

### Tailwind passthrough

- `tailwind/base/transparent` = #ffffff00 — used for the `border-2 transparent` breathing margin on Default/Disabled track

## Spacing

`spacing/0-5` (2px) · `spacing/2` (8px) · `spacing/2-5` (10px, **NEW**) · `spacing/3` (12px, **NEW**)

- spacing/2 → container gap (track to label area)
- spacing/0-5 → fieldContent gap (label to description)
- spacing/2-5 → Box-type container padding
- spacing/3 → reserved, not used by Switch (carries over from token pool)

## Height + Width

`height/h-3` (12px, **NEW**) · `height/h-4` (16px)
`width/w-3` (12) · `width/w-4` (16) · `width/w-6` (24) · `width/w-8` (32, **NEW**)

- h-4/w-4 → default-size thumb (16×16)
- h-3/w-3 → sm-size thumb (12×12)
- w-8 → default-size track width
- w-6 → sm-size track width

## Border-radius

- `border-radius/rounded-full` = 9999px — track + thumb (universal pill shape)
- `border-radius/rounded-lg` = 10px (**NEW**) — Box-type outer container

## Border-width

- `border-width/border` = 1px — Box container border + Focus track border + Invalid track border
- `border-width/border-2` = 2px (**NEW**) — Default/Disabled track breathing margin (transparent)

## Opacity

- `opacity/opacity-50` = 0.5 (**NEW**) — Disabled state on outer container. Cross-component inconsistency with Button (which uses tinted color). Audit Finding 8.

## Typography

**Atomic**

- Family: `font.font-sans` = Open Sans
- Weights: `font-weight.medium` 500 · `font-weight.normal` 400 (**NEW**)
- Text scale: `sm` (14px / 20px line-height)

**Compound** (all NEW vs Button bundle)

- `typography.text-sm-leading-normal-medium` — Label (default-size)
- `typography.text-sm-leading-normal-normal` — Description (all sizes)
- `typography.text-sm-leading-none-medium` — Label (sm-size, line-height: 1 — Apollo v2 packs sm tighter)

## Effects

- `effect.focus-default` — outset drop-shadow, 3px spread, `{color.custom.outline}`. Switch Focus state ring.
- `effect.focus-destructive` — outset drop-shadow, 3px spread, `{color.custom.destructive-20-dark-40}`. Switch Invalid state ring — **shown at REST**, not only on Focus.

## Raw values (audit signals, not tokenized)

- Track height default-size: **18.4px raw** (F9 audit) — no `height/h-*` token matches
- Track height sm-size: **14px raw** (F10 audit) — same class as F9
- Container widths: **312/304/300/296 raw** — hardcoded across all walked cells
- Track padding-x: **1px raw** — no `spacing/*` token at this value

## How to resolve a binding

Same as Button bundle. `{path.to.token}` → nested key path in `tokens.json`. Examples:

- `{color.base.primary}` → `#5a35c0`
- `{border-radius.rounded-full}` → `9999px`
- `{typography.text-sm-leading-none-medium}` → composite (Open Sans / Medium / 14px / 1.0 / 0px)

If a `$value` is itself a `{…}` reference, follow until a leaf.
