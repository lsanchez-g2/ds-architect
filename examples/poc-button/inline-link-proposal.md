# InlineLink — Proposal for Apollo v2

> **Status:** specification-only proposal · v0.1.0 (DRAFT)
> **Targets:** Apollo v2 (SA) Design System — Figma file `3401ZFUHoboOwA6GGjAEsq`
> **Origin:** `audit-findings-for-source.md` Finding 1 (HIGH severity) — Button.Link breaks inline-in-body-text usage.
> **Decision (2026-05-15):** Option A from the audit memo — add a dedicated `InlineLink` component. Spec lives here until ratified + built in Figma by the Apollo v2 maintainer post-OOO.

---

## 1. Why this exists

Apollo v2 Button.Link variant has typography hard-bound to `{typography.text-lg-leading-normal-semibold}` (Open Sans / SemiBold / 18px / 28px). That works for standalone CTA-style links sized for button rows. It **breaks** when designers reach for it inline in a paragraph because body text is 16px / Regular and the link visibly jumps a step in both size and weight.

The Button.Link variant was never intended for inline usage, but it's the only "link-looking" thing in the system, so it gets misused. Adding a dedicated `InlineLink` removes the misuse path.

This proposal does NOT modify Button.Link. Button.Link stays as-is for standalone CTA usage. InlineLink is a new, separate atom for body-flow usage.

---

## 2. Scope

### In scope
- New `InlineLink` component with state matrix (Default, Hover, Focus, Disabled, Pressed, optionally Visited)
- Typography that **inherits** from parent (no fixed size/weight/line-height)
- Underline always applied (text-decoration on the TEXT node per SP-3, not via underlined typography token)
- Color tokens from existing palette (no new tokens required for the base case)
- Focus ring via existing `{effect.focus-default}`
- Accessibility: native `<a>` role, focus-visible, motion-reduce, sufficient contrast in both modes
- A bundle-format `InlineLink.component.json` skeleton (embedded below) ready for extraction once the component exists in Figma

### Out of scope (initially)
- Optional icon slots (external-link `↗`, trailing arrow). Defer to v0.2.0 of InlineLink if needed.
- A `size=inline` shim on Button.Link. Audit memo Option B explicitly rejected.
- Dark mode. Apollo v2 bundle is light-only at the moment; InlineLink dark mode comes when the rest of the DS adds dark mode tokens.
- Visited state. Optional in v0.1.0; add as separate variant in v0.2.0 if Apollo v2 wants to track read history.

---

## 3. Component shape

### 3.1 Variant axes

Single axis: `state`.

| state | Visual treatment |
|---|---|
| Default | text color = brand, underlined |
| Hover | text color = brand-hover, underlined |
| Focus | text color = brand (Default), underlined, **+ focus ring** |
| Disabled | text color = primary-disabled, underlined, no pointer events |
| Pressed | text color = brand-active, underlined |

No `size` axis. Size inherits from the parent text context.
No `variant` axis. There is exactly one InlineLink visual identity. If you need a second style, it's a different component.

Total cells: **5** (vs Button's 264). Tiny matrix on purpose — inline links should not have configurable visual weight; they should always read as the same affordance.

### 3.2 Slots

| Slot | Type | Default | Notes |
|---|---|---|---|
| `label` | TEXT | "link" | Visible link text. Typography **inherits** from parent. Apply `textDecoration: UNDERLINE` on the TEXT node per SP-3. |

No leftIcon / rightIcon / kbdGroup slots in v0.1.0. If you need them later, add as exposed boolean slots like Button does.

### 3.3 Layout

- **No fixed dimensions.** Width and height inherit; the link is a `display: inline` element.
- **No padding.** Inline-flow elements don't get padding; surrounding text rhythm handles spacing.
- **No border-radius.** This is text, not a pill. The pill convention is Button's identity, not InlineLink's.
- **No background fill.** Transparent. Bind to `{color.tailwind.base.transparent}` for explicit binding (per SP-4 fully-bound > hardcoded).

### 3.4 Bindings (per state)

| Property | Default | Hover | Focus | Disabled | Pressed |
|---|---|---|---|---|---|
| `text-color` | `{color.base.primary}` | `{color.base.primary-hover}` | `{color.base.primary}` | `{color.base.primary-disabled}` | `{color.base.primary-active}` |
| `text-decoration` | UNDERLINE | UNDERLINE | UNDERLINE | UNDERLINE | UNDERLINE |
| `text-decoration-thickness` | 1px | 1px | 1px | 1px | 1px |
| `text-underline-offset` | 2px | 2px | 2px | 2px | 2px |
| `focus-ring` | n/a | n/a | `{effect.focus-default}` | n/a | n/a |
| `cursor` | pointer | pointer | pointer | not-allowed | pointer |
| `pointer-events` | auto | auto | auto | none | auto |
| `typography.fontFamily` | inherit | inherit | inherit | inherit | inherit |
| `typography.fontSize` | inherit | inherit | inherit | inherit | inherit |
| `typography.fontWeight` | inherit | inherit | inherit | inherit | inherit |
| `typography.lineHeight` | inherit | inherit | inherit | inherit | inherit |

**Critical:** the typography fields say `inherit`, not a token reference. This is the difference between InlineLink and Button.Link. A renderer reading this bundle MUST honor `inherit` — do NOT substitute a default typography token.

### 3.5 No new tokens needed

All 5 color values exist already in `tokens.json`:

- `base/primary` (#5a35c0)
- `base/primary-hover` (#4a2ba3)
- `base/primary-active` (#3b2185)
- `base/primary-disabled` (#a78bfa) — **note:** see §4 below on the contrast caveat
- `tailwind/base/transparent` (#ffffff00) for explicit fill

Focus ring: `{effect.focus-default}` (existing — 3px outset drop-shadow at `{color.custom.outline}`).

If Visited state is added in v0.2.0, propose adding a new `base/primary-visited` token (e.g. #6b46c1) rather than reusing `muted-foreground`. Optional.

---

## 4. Accessibility

### 4.1 Native semantics

- Render as `<a href="…">` when the link is a real navigation/external link.
- Use `<button type="button" class="inline-link">` only when the affordance is "do something" rather than "go somewhere" and styling needs to match. Prefer `<a>` whenever possible.
- `role` is inferred from the element; do not add `role="link"` to a real `<a>`.

### 4.2 Focus visibility

- Use `:focus-visible`, not bare `:focus`. The focus ring should appear on keyboard focus, not mouse focus.
- Focus ring = `{effect.focus-default}` (outset drop-shadow). Outline-style focus would conflict with the underline — keep the drop-shadow convention to match Button family.

### 4.3 Contrast caveat — Disabled state

`{color.base.primary-disabled}` = #a78bfa on a typical body-text background (#fafafa) measures **2.49:1** — below WCAG AA's 4.5:1 for body text.

**This is acceptable for disabled UI** (WCAG SC 1.4.3 explicitly exempts inactive UI components), but flag it in the spec: disabled inline links are intentionally low-contrast as a visual cue. Don't use disabled inline links for content the user is expected to read.

If a higher-contrast disabled treatment is desired, propose a new `base/inline-link-disabled` token at ~4.6:1 against #fafafa (e.g. #7c5db5 or similar).

### 4.4 Motion-reduce

No animations on the base InlineLink (no spinner like Button.Loading). Future enhancements (hover micro-motion, etc.) should respect `prefers-reduced-motion` per the candidate SP-9 spec patch.

### 4.5 Touch target

Inline links live in body-text contexts. They cannot be 44×44 because that would break the line. WCAG SC 2.5.5 allows smaller targets when "the target is in a sentence or block of text". InlineLink is exactly this case — exempt.

Don't try to inflate inline links into touch-target compliance. That's how you end up with Button.Link inline in the first place.

---

## 5. Bundle-format entry (skeleton)

Sketch of `data/components/InlineLink.component.json` for when the component exists in Figma:

```json
{
  "$schema": "../../../../BUNDLE_SPEC.md#section-6.2",
  "$bundleSchemaVersion": "0.2.0",
  "$proposed": true,
  "$proposalRef": "examples/poc-button/inline-link-proposal.md",
  "$proposalStatus": "specification-only — not yet authored in Figma",

  "id": "inline-link",
  "name": "InlineLink",
  "level": "atom",
  "figmaComponentSetId": "TBD",
  "figmaFileKey": "3401ZFUHoboOwA6GGjAEsq",
  "description": "Inline anchor for body-text usage. Typography inherits from parent context. Underline is the affordance.",
  "documentationLinks": [
    "audit-findings-for-source.md#finding-1-high-impact",
    "inline-link-proposal.md"
  ],

  "variantProperties": {
    "state": {
      "type": "VARIANT",
      "values": ["Default", "Hover", "Focus", "Disabled", "Pressed"]
    }
  },
  "variantConstraints": [],
  "totalVariantCells": 5,

  "defaultVariant": { "state": "Default" },

  "exposedProps": [
    { "name": "label",  "type": "TEXT", "default": "link", "description": "Visible link text." },
    { "name": "href",   "type": "TEXT", "default": "#",    "description": "Target URL when rendered as <a>." }
  ],

  "slots": [
    {
      "name": "label",
      "type": "TEXT",
      "typography": "inherit",
      "$typographyNote": "Typography MUST inherit from parent context. Do not substitute a default typography token. SP-3 textDecoration captured separately on the TEXT node."
    }
  ],

  "layout": {
    "mode": "INLINE",
    "$inlineNote": "InlineLink is display:inline. No frame, no fixed dimensions, no padding, no border-radius. Inherits width/height from text flow."
  },

  "bindings": {
    "radius":  { "$all": "none" },
    "opacity": { "$all": "{opacity.opacity-100}" },
    "fill":    { "$all": "{color.tailwind.base.transparent}" },
    "border":  { "$all": "none" },

    "text-color": {
      "Default":  "{color.base.primary}",
      "Hover":    "{color.base.primary-hover}",
      "Focus":    "{color.base.primary}",
      "Disabled": "{color.base.primary-disabled}",
      "Pressed":  "{color.base.primary-active}"
    },

    "text-decoration": {
      "$all": "UNDERLINE",
      "thickness": "1px",
      "underlineOffset": "2px",
      "$captureNote": "SP-3 (v0.2.0): textDecoration MUST be emitted on the TEXT node independently of any typography token. Renderer applies as CSS text-decoration: underline; text-underline-offset: 2px; text-decoration-thickness: 1px."
    },

    "focus-ring": {
      "Focus": "{effect.focus-default}"
    },

    "cursor": {
      "Default":  "pointer",
      "Hover":    "pointer",
      "Focus":    "pointer",
      "Disabled": "not-allowed",
      "Pressed":  "pointer"
    }
  },

  "composition": { "uses": [] },

  "a11y": {
    "role": "link",
    "preferredElement": "<a href>",
    "fallbackElement": "<button type=button class=inline-link>",
    "focusRing": "{effect.focus-default}",
    "focusSelector": ":focus-visible",
    "minTouchTarget": null,
    "$minTouchTargetNote": "WCAG SC 2.5.5 exempts inline-text targets. Do NOT inflate to 44x44 — it would break line rhythm and reintroduce the Button.Link inline misuse problem.",
    "contrastNote": "Disabled state intentionally low-contrast (~2.5:1) per WCAG SC 1.4.3 inactive-UI exemption. Do not use Disabled inline links for content the user is expected to read."
  },

  "codeConnect": {
    "candidates": [
      {
        "framework": "react",
        "library": "shadcn/ui",
        "$note": "shadcn/ui has no separate InlineLink component. Closest equivalent is the project-level `<a>` element styled with Tailwind utilities. Apollo v2 can ship a `<InlineLink>` React wrapper at e.g. src/components/ui/inline-link.tsx."
      }
    ]
  },

  "$openQuestions": [
    "Should v0.1.0 ship with a Visited state? (Currently no.)",
    "Do we add base/primary-visited token now or wait for first real use case?",
    "Should the underline-offset and thickness be tokenized (text-decoration-thickness/underline-offset tokens) or stay inline?",
    "Light + dark — when Apollo v2 ships dark mode, what is the dark-mode link color?"
  ]
}
```

---

## 6. Figma authoring guidance (for Apollo v2 maintainer)

When implementing this in Figma (post-OOO):

### 6.1 Component-set structure

Create a new component-set on the Apollo v2 Components page (sibling to Button), named `InlineLink`. One variant axis: `State` with 5 values (Default, Hover, Focus, Disabled, Pressed).

### 6.2 Per-cell node tree

Each cell = one TEXT layer.

- **Auto-layout:** off (or HORIZONTAL with hug × hug — both work for a single text child).
- **Frame size:** hug content both axes.
- **Frame fill:** none. **Frame stroke:** none. **Frame corner-radius:** 0.
- **TEXT layer:**
  - Characters: "link"
  - Font: **do not set a Figma text style** — leave font family / size / weight / line height **at defaults**. The whole point is that this text inherits from the parent.
  - Text decoration: **Underline** (via the TEXT layer's text-decoration toggle, not via a typography variable)
  - Fill: per the state table in §3.4

### 6.3 Variable bindings to apply

Bind the TEXT layer fill to:

- Default → `base/primary`
- Hover → `base/primary-hover`
- Focus → `base/primary`
- Disabled → `base/primary-disabled`
- Pressed → `base/primary-active`

On the Focus cell, add an Effect: drop-shadow with `focus/default` variable. No effects on other cells.

### 6.4 Component description

Set the Figma component description to:

> Inline anchor for body-text usage. Typography inherits from parent context. Use Button.variant=Link only for standalone CTA usage.

Link the documentationLink to whatever location Apollo v2 hosts component docs.

### 6.5 What NOT to do

- ❌ Don't bind the TEXT to a typography token (`text-lg-leading-normal-semibold` etc.). That reintroduces the size-mismatch bug.
- ❌ Don't add a frame fill or radius. This is text, not a pill.
- ❌ Don't add icon slots in v0.1.0. Keep it minimal until there's a confirmed use case.
- ❌ Don't add a `size` axis. The whole point is no size variation.

---

## 7. Validation plan (when component exists)

1. Walk the new Figma component-set via Figma MCP, just like Button.
2. Emit `data/components/InlineLink.component.json` (real, not proposed) + `data/components/InlineLink.variants.json` (5 cells).
3. Add bundle entry to `MANIFEST.json.counts` (`components: 2`, `variantCells: 269`).
4. Re-run smoke test in Claude Design with a prompt like *"Render this paragraph using Apollo v2 InlineLink for the highlighted text"* — assert that the rendered link has the same font-size as the surrounding body text.
5. Compare against the Button.Link inline misuse case from smoke test #1 to confirm the line-rhythm break is fixed.

Acceptance gate: in a 16px regular paragraph, the rendered InlineLink reads at 16px regular (inherited) with underline + primary color. No size or weight jump.

---

## 8. Token additions deferred

Two token additions are CANDIDATE for v0.2.0 of this proposal but NOT required for v0.1.0:

- `base/primary-visited` — if Apollo v2 wants to track read history. Suggested value: #6b46c1 (between primary and primary-hover).
- `base/inline-link-disabled` — if §4.3 contrast is unacceptable. Suggested: #7c5db5 or similar at ~4.6:1 vs #fafafa.

Neither is needed for the base implementation. Add when use case lands.

---

## 9. Status + ownership

- **Owner:** lsanchez-g2 (Apollo v2 maintainer)
- **Status:** Specification-only. Not yet authored in Figma. Not yet extracted into a real bundle entry.
- **Blockers:** none (proposal can land in Figma anytime post-OOO).
- **Effort estimate:** 1–2 hours in Figma to author the 5-cell component-set + bind variables + add description. Plus re-running the extraction pipeline once the component is live.
- **Resume:** when post-OOO Apollo v2 work picks up, this doc is the implementation brief.

---

## 10. Why this approach was chosen

Option A (new component) over Option B (size=inline on Button.Link) because:

1. **Conceptual cleanliness.** Buttons are standalone interactive controls with fixed dimensions, pill shape, type scale. Inline anchors are text fragments inheriting their context. Conflating the two muddies both.
2. **No silent breakage.** A `size=inline` shim on Button.Link would let designers keep reaching for Button.Link inline — just with one more knob to remember. A separate component routes them to the right primitive.
3. **Cleaner shadcn parity.** shadcn/ui has a separate `<a>` styling vs Button.variant=link. Apollo v2's two-component model maps cleanly.
4. **Smaller surface to extract.** 5 cells vs would-be 24 cells (size=inline × Link × 6 states × 4 sizes).
5. **Easier to deprecate** if Apollo v2 ever wants to retire Button.Link entirely. Two separate components ratchet cleanly; a shim doesn't.

Option C (both) was offered but adds migration burden without solving the conceptual issue. Skip.
