# Patterns — Apollo v2 Switch

> Cross-cell + system-level conventions surfaced during Switch PoC extraction. Companion to `examples/poc-button/references/patterns.md`.

## 1. Active-axis is layout-driven, not position-driven

Track is a HORIZONTAL flex container with one child (the thumb). `justify-content` shifts between `flex-start` (Active=Off) and `flex-end` (Active=On). Smart-animate slides between the two layout endpoints.

→ No `position: absolute`, no `transform: translate(…)`. Clean CSS. Easy to honor `prefers-reduced-motion: reduce` via SP-9 policy=`skip` (instant variant change).

## 2. Track-border breathing margin (Default/Disabled)

Default and Disabled track use `border: 2px solid transparent` even though there's no visible border. This is a **breathing margin pattern**: the transparent border reserves 2px around the thumb so it never visually collides with the track edge.

Consumers must preserve this — don't strip the transparent border thinking it's a no-op. The thumb's effective horizontal travel range = `track-width − thumb-width − 2 × border-spacing`.

Focus and Invalid states swap this breathing margin for a 1px solid colored border (ring on Focus, destructive on Invalid).

## 3. Disabled = container-level opacity, NOT per-element

`opacity: 0.5` is applied at the **outer `<button>` container** for Disabled. It dims the entire component — track, thumb, label, description — uniformly.

→ Don't apply `opacity: 0.5` to inner elements individually. Doing so compounds the dim (50% × 50% = 25%) and breaks the intended affordance.

→ Cross-component inconsistency with Button (which uses tinted color for Disabled, no alpha). F8 audit signal.

## 4. Invalid state shows the destructive ring at REST

Apollo v2 Switch Invalid state emits the `focus-destructive` drop-shadow **always**, not only on Focus. F11 — apparently intentional Apollo v2 design: invalidness is signaled at rest, not just when the component is focused.

→ Renderers honor faithfully. Documented so consumers don't think it's a Focus-only effect.

## 5. State-axis is incomplete vs Button

Switch state axis = `{Default, Disabled, Focus, Invalid}` (4 values).
Button state axis = `{Default, Hover, Focus, Loading, Disabled, Pressed}` (6 values).

→ Switch lacks Hover, Pressed, Loading. Audit signals F5/F6/F7. Source-DS owner decides whether to fill the gaps.

Practical impact:
- No mouse-hover feedback on Switch track or thumb
- No active-press visual during toggle interaction
- No async-toggle busy state (Switch doesn't have a Spinner-swap pattern like Button.Loading)

## 6. Type=Box wraps Type=Default in a frame

Type=Box adds a wrapping container with:
- 1px solid `{color.base.border}` border
- `{spacing.2-5}` (10px) padding
- `{border-radius.rounded-lg}` (10px) corner radius
- `{color.base.background}` fill

The inner contents (track + thumb + label + description) are **identical to Type=Default**. Box is purely a frame wrapper for form-field placement parity.

→ Renderers: implement Box as `<Switch>` wrapped in a styled `<div>`, not as a separate component.

## 7. sm-size packs typography tighter

Label typography is size-dependent:

- `default` → `text-sm-leading-normal-medium` (font-size 14 / line-height 20)
- `sm`      → `text-sm-leading-NONE-medium`   (font-size 14 / line-height 1)

Same font-size, tighter line-height on sm. Description typography is constant across sizes.

→ When rendering sm cells, use `line-height: 1` on the label specifically. Don't reuse the default-size typography token.

## 8. Track height is non-tokenized

Track height: 18.4px (default-size) and 14px (sm-size). Both are **raw values, not bound to any height token**. Apollo v2 lacks `height/h-4-5` (18px or 18.4px) and `height/h-3-5` (14px) tokens.

F9 + F10 audit signals. Source-DS owner decides: add the tokens OR document the raw values as intentional.

## 9. Container widths are fixed, hardcoded

All Switch cells emit fixed container widths: 312px (Type=Default default), 304px (Type=Default sm), 300px (Type=Box default), 296px (Type=Box sm).

None bound to a width token. Likely intentional (Switch is sized for specific form-row contexts), but worth flagging — same class of audit signal as Button's container widths.

## 10. SP-9 motionReduce policy = `skip`, not `slow`

Switch's toggle animation is a **state transition** (Off ↔ On). Reduced-motion users get policy=`skip`: instant variant change, no slide.

This is different from Button's Spinner (looping animation) which uses policy=`slow` (4× duration, preserves the activity signal).

Together they validate SP-9 across both motion patterns: looping (Button Spinner) and state-transition (Switch toggle).

## 11. aria-checked + aria-invalid are renderer responsibilities

Apollo v2 source DS encodes the visual states (Active=Off/On + State=Invalid) but doesn't write `aria-*` attributes directly in Figma. The bundle promises:

- Off → renderer emits `aria-checked="false"`
- On → renderer emits `aria-checked="true"`
- Invalid → renderer emits `aria-invalid="true"`

Same contract pattern as Button.Loading → `aria-busy="true"` (renderer fulfills the promise the bundle declares).

## 12. ControlPlacement is layout-only, no visual identity change

Switch can have its track on the Start (left) or End (right) of the label area. The axis only flips the item order in the horizontal container. No visual identity change.

→ Renderers can implement as a single CSS class flip (`flex-direction: row` vs `row-reverse`), or as an item-order property on the auto-layout container. Both produce the same result.

## 13. Source-DS findings are part of the bundle

Same convention as Button bundle. F5–F11 documented in `Switch.component.json` + here in §5/§3/§4/§8. Renderers surface findings rather than rendering around them. The bundle reflects source-DS reality; the renderer doesn't paper over.
