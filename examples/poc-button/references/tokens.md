# Tokens — Quick Reference

> Human-readable summary of `data/tokens.json` (84 leaves). Source: Apollo v2 (SA) Design System file `3401ZFUHoboOwA6GGjAEsq`, Button-subtree scope.
>
> For per-token `$value` / `$type` / `$description` / `$extensions`, load `data/tokens.json` directly.

## Colors — Semantic (base/*)

Variant action palette: every variant has matching default/hover/active/disabled + foreground pairs.

| Family | default | hover | active | disabled |
|---|---|---|---|---|
| primary     | `#5a35c0` violet | `#4a2ba3` | `#3b2185` | `#ede9fe` (tinted) |
| secondary   | `#e1ff6d` lime   | `#f7ffcc` | `#eeff99` | `#f7ffcc` |
| destructive | `#dc2626` red    | `#b91c1c` | `#b91c1c` | `#fee2e2` (tinted) |

Plus: `accent` (#ede9fe / #5a35c0 fg), `muted-foreground` (#737373), `muted` (#f5f5f5), `background` (#fafafa).

## Colors — Custom (alpha + dark-mode-aware)

- `outline` = `#a3a3a380` — focus-ring base (neutral 50% alpha)
- `destructive-10-dark-20` = `#dc26261a` — destructive focus ring base
- `destructive-20-dark-40` = `#dc262633` — destructive focus ring spread
- `bg-background-20-dark-bg-background-10` = `#ffffff33` — translucent KbdGroup background
- `background-dark-input-30` = `#fafafa`

## Colors — Tailwind passthrough

- `tailwind.red.300` = `#fca5a5`
- `tailwind.base.transparent` = `#ffffff00`

## Spacing (base unit 4px)

`spacing.0-5` 2px · `spacing.1` 4px · `spacing.1-5` 6px · `spacing.2` 8px · `spacing.4` 16px · `spacing.6` 24px · `spacing.8` 32px · `spacing.12` 48px

## Height

`h-4` 16 · `h-6` 24 · `h-9` 36 · `h-10` 40 · `h-11` 44 · `h-12` 48

## Width

`w-3` 12 · `w-4` 16 · `w-6` 24 · `w-11` 44

## Border-radius

- `rounded-full` = **9999px** — pill. Universal on Button across all 264 cells. Canonical fully-rounded round-trip test target.
- `rounded-sm` = 6px — Kbd inner chip.

## Border-width

- `border` = 1px (default border weight on Outline variant)

## Stroke-width (Lucide icon strokes)

`stroke-1` 1 · `stroke-1-33` 1.33 (Lucide default) · `stroke-1-5` 1.5 · `stroke-2` 2

## Opacity

`opacity-100` = 1

## Typography

**Atomic**

- Family: `font.font-sans` = `Open Sans`
- Weights: `font-weight.semibold` 600 · `font-weight.medium` 500
- Text scales (font-size / line-height pairs): `sm` (14/20) · `base` (16/24) · `lg` (18/28) · `xl` (20/28)

**Compound** (each composes family + weight + size + line-height + letterSpacing)

- `text-sm-leading-normal-semibold`
- `text-base-leading-normal-semibold`
- `text-lg-leading-normal-semibold`
- `text-xl-leading-normal-semibold`
- `text-sm/base/lg/xl-leading-normal-underlined-semibold` — present in source but NOT applied by Button.Link variant (textDecoration is applied separately per SP-3 v0.2.0).

## Effects

- `effect.focus-default` — outset drop-shadow, 3px spread, color `{color.custom.outline}`. Applied to non-Destructive variants on `state=Focus`.
- `effect.focus-destructive` — outset drop-shadow, 3px spread, color `{color.custom.destructive-20-dark-40}`. Applied to Destructive variant on `state=Focus`.

## Icon-set marker

- `icon-set.lucide-icons` = `true` (boolean marker; declares Apollo v2 Button uses Lucide).

## Modes

This bundle ships **light mode only**. Dark mode values not yet extracted.

## Spec patches relevant to tokens

- SP-8 — nested-instance contributions: 5 tokens were added by walking the nested KbdGroup → Kbd descendant: `base/muted`, `base/background`, `border-radius/rounded-sm`, `font-weight/medium`, `custom/bg-background-20-dark:bg-background-10`. The extractor MUST walk nested INSTANCEs, not just `get_variable_defs` on the parent subtree.

## How to resolve a binding

`{path.to.token}` references in component / variant files map to a nested key path in `tokens.json`. Examples:

- `{color.base.primary}` → `tokens.json[color][base][primary].$value` = `#5a35c0`
- `{border-radius.rounded-full}` → `9999px`
- `{typography.text-lg-leading-normal-semibold}` → composite (family + weight + size + line-height; references atomic tokens via `{font.font-sans}` etc.)

If a `$value` is itself a `{…}` reference, follow until you reach a leaf.
