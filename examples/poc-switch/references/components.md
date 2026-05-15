# Switch — Quick Reference

> Summary of `data/components/Switch.component.json` + `Switch.variants-samples.json`. For full machine-readable schemas, load the JSON files directly.

## Identity

- **Component:** Apollo v2 Switch
- **Figma component-set node:** `60:450` (in file `3401ZFUHoboOwA6GGjAEsq`)
- **Atomic level:** atom
- **shadcn parity:** maps to https://ui.shadcn.com/docs/components/switch (with Apollo v2 extensions: size axis + Type=Box variant)

## Variant matrix — 64 cells

5 axes:

| Axis | Values | Count |
|---|---|---|
| **size** | `default` (42h), `sm` (36h) | 2 |
| **active** | `Off`, `On` | 2 |
| **state** | `Default`, `Disabled`, `Focus`, `Invalid` | 4 |
| **type** | `Default`, `Box` | 2 |
| **controlPlacement** | `Start`, `End` | 2 |

Total = 2 × 2 × 4 × 2 × 2 = **64**.

**Missing states vs Button:** no Hover, no Pressed, no Loading. Audit signals F5/F6/F7.

## Exposed props

| Prop | Type | Default |
|---|---|---|
| `labelText` | TEXT | "Switch Text" |
| `descriptionText` | TEXT | "This is a switch description." |
| `showLabel` | BOOLEAN | true |
| `showDescription` | BOOLEAN | true |

No size/state/active/type/controlPlacement props — those are variant-axis enums.

## Slots

- **track** — pill-shaped toggle background. Color changes by (active × state). First-class slot.
- **thumb** — round circle inside track. Fill constant (#fafafa); position shifts by active.
- **label** — TEXT, exposed by `showLabel`.
- **description** — TEXT, exposed by `showDescription`.

## Bindings overview

### track-fill (axisDependency: active + state)

| | Default | Disabled | Focus | Invalid |
|---|---|---|---|---|
| Off | input-dark-input-80 | input-dark-input-80 | input-dark-input-80 | input-dark-input-80 |
| On  | primary             | primary              | primary              | destructive          |

→ Off track-fill is **state-invariant** (always the gray). State changes Off track via border + shadow + opacity, not fill. On track-fill goes destructive only when Invalid.

### thumb-fill

`{color.base.background}` — constant across all 64 cells.

### thumb-position (axisDependency: active)

- Off → track `justify-content: flex-start` (thumb at left)
- On  → track `justify-content: flex-end` (thumb at right)

Layout-driven. Smart-animate slides between the two endpoints. Animation block carries `motionReduce.policy: skip` (SP-9).

### track-border (axisDependency: state)

| State | Border |
|---|---|
| Default  | border-2 transparent (breathing margin) |
| Disabled | border-2 transparent (same as Default) |
| Focus    | border-1 base/ring (#5a35c0) |
| Invalid  | border-1 base/destructive (#dc2626) |

### track-shadow / focus-ring (axisDependency: state)

| State | Shadow |
|---|---|
| Default  | none |
| Disabled | none |
| Focus    | `{effect.focus-default}` |
| Invalid  | `{effect.focus-destructive}` **AT REST** (always, not only on Focus — F11) |

### opacity (axisDependency: state, applied at outer container)

| State | Opacity |
|---|---|
| Default  | 1 |
| Disabled | `{opacity.opacity-50}` — applied at the OUTER `<button>` container, dims entire component |
| Focus    | 1 |
| Invalid  | 1 |

→ F8: Inconsistent with Button (which uses tinted color, no alpha).

### label fill

| State | Color |
|---|---|
| Default  | base/foreground (#0a0a0a) |
| Disabled | base/foreground (dimmed via container opacity) |
| Focus    | base/foreground |
| Invalid  | base/destructive (#dc2626) |

### description fill

`{color.base.muted-foreground}` (#737373) — **constant across ALL states including Invalid** (F11 corollary).

### size-axis bindings

| size | trackWidth | trackHeight | thumbSize | labelTypography |
|---|---|---|---|---|
| default | w-8 (32) | 18.4px raw (F9) | 16×16 | text-sm-leading-normal-medium (14/20) |
| sm | w-6 (24) | 14px raw (F10) | 12×12 | text-sm-leading-NONE-medium (14/1) |

Apollo v2 packs sm tighter via line-height: 1.

### type-axis bindings

- **Default**: container 312/304px, no border, no padding, no radius, no fill — bare inline layout
- **Box**: container 300/296px, 1px solid base/border, padding spacing/2-5 (10px), rounded-lg radius, base/background fill

### controlPlacement-axis (layout-only)

- **Start**: `[track, fieldContent]` order
- **End**: `[fieldContent, track]` order — flips item order in horizontal container; no visual identity change

## Animation (SP-9)

```
trigger: ON_CLICK
action:  CHANGE_VARIANT axis=active
transition: SMART_ANIMATE, duration ≈ 150ms, easing ease-out (Step 5 unconfirmed)
motionReduce: policy=skip, $reason="state-transition animations should jump to end state"
```

Validates SP-9 across both motion patterns when combined with Button:
- Button.Loading Spinner → policy=`slow` (looping animation)
- Switch.active=Off↔On → policy=`skip` (state transition)

## A11y

- `role="switch"`
- `aria-checked`: "true" when Active=On, "false" when Off
- `aria-invalid="true"` on Invalid state
- Disabled: `disabled` + `aria-disabled="true"` (renderer responsibility)
- Focus ring per binding above. `:focus-visible` only, not bare `:focus`.
- Touch-target compliance: only Type=Box passes WCAG 2.5.5. Type=Default cells fail (42h/36h).

## Code Connect (shadcn parity)

Maps to shadcn Switch via `checked` prop. Apollo v2 adds:
- `sm` size (shadcn has one size)
- `Box` type (shadcn doesn't)
- `controlPlacement` (shadcn-side concern is layout, not the component)

## Audit findings ledger (for Apollo v2 source-DS work)

| ID | Severity | Finding |
|---|---|---|
| F5 | MEDIUM | Switch lacks Hover state |
| F6 | LOW | Switch lacks Pressed state |
| F7 | LOW | Switch lacks Loading state |
| F8 | MEDIUM | Disabled uses opacity-50 (Switch) vs tinted color (Button) — cross-component inconsistency |
| F9 | MEDIUM | Track height 18.4px raw on default-size (no token at this value) |
| F10 | MEDIUM | Track height 14px raw on sm-size (no token at this value) |
| F11 | MEDIUM-doc | Invalid state shows destructive ring at REST — intentional emphasis, documented for downstream consumers |

Plus: container widths (312/304/300/296) all hardcoded raw across all walked cells.
