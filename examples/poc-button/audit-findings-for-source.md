# Apollo v2 Audit Findings — Surfaced by PoC

> Source DS: **Apollo v2 (SA) - Design System**
> Figma file: `3401ZFUHoboOwA6GGjAEsq`
> Component: Button (node `37:931`)
> Surfaced during: ds-architect v3 lossless-extraction PoC (Steps 0–3)
> Date: 2026-05-15
> Owner: lsanchez-g2 (Apollo v2 maintainer, OOO May 26 → Sep 1)

These are audit findings against the **source design system**, not bugs in the bundle or the extractor. The bundle faithfully captures what Apollo v2 currently defines. If the source is wrong, the bundle reflects that — which is correct behavior for a lossless extractor, but actionable for the DS owner.

Three findings, ranked by impact.

---

## Finding 1 (HIGH IMPACT) — Button.Link variant breaks when used inline in a sentence

### Symptom

When `Button` variant=Link size=default is placed inside a running sentence, the link text renders **larger and heavier than the surrounding body text**, creating a visible mismatch.

Reverse-render evidence in Claude Design smoke test (2026-05-15):

> "Your changes are saved automatically. **View history** to review prior versions."

The "View history" inline link rendered ~2px taller than the body text, and was visibly heavier (semibold vs. regular). The pill-button context is missing; the underlined text was the only affordance — yet the type styling was Button-row-scale.

### Root cause

Apollo v2 `Button.variant=Link` size=default has typography hard-bound to:

```
{typography.text-lg-leading-normal-semibold}
  = Open Sans / SemiBold / 18px / line-height 28px
```

Standard body copy in Apollo v2 contexts (where inline links naturally appear) is:

```
text-base / Regular / 16px / line-height 24px
```

→ 18px vs 16px (font-size mismatch) + Semibold (600) vs Regular (400) (weight mismatch).

The Button.Link variant was designed for **standalone CTA-style links** (sitting next to other Buttons, at Button-row scale). It is NOT designed for inline-in-text usage.

But the component name "Link" reads as a generic "any link". Designers and AI consumers (Claude Design) reach for it whenever they need an underlined inline anchor — because it's the only thing in the DS labeled "Link".

### Impact

- **Designers using Button.Link inline** get unintended visual weight + size jump. Body text reads broken.
- **Claude Design (and any LLM consumer)** picks Button.Link for inline anchors because the bundle exposes no distinction. Every reverse-render of inline-link-in-prose will fail this same way.
- **Accessibility:** mid-sentence type-size jumps are flagged by WCAG-aware reviewers as line-rhythm disruption.

### Recommended fix (two options)

**Option A (preferred) — Add a dedicated `InlineLink` component.**

Separate primitive. Lives in body-text contexts. Typography inherits from parent (`font-size: inherit; font-weight: inherit; line-height: inherit`). Only its own visual identity: color (`base/primary`), textDecoration UNDERLINE, hover/focus/disabled states matching Button.Link's color scheme.

Why preferred: clean conceptual model. Button.Link stays a Button (standalone CTA). InlineLink is unambiguously for inline use. Both can exist in the system.

Rough spec:
```
InlineLink
  states: Default, Hover, Focus, Disabled, Visited (optional)
  typography: inherit (no fixed size/weight)
  textDecoration: UNDERLINE always
  text-color:
    Default: {color.base.primary}
    Hover:   {color.base.primary-hover}
    Focus:   {color.base.primary} + {effect.focus-default}
    Disabled:{color.base.primary-disabled}
    Visited: {color.base.muted-foreground} (optional)
  cursor: pointer
  underline-offset: ~2px (tunable)
  underline-thickness: 1px or inherit
```

**Option B — Add a `size=inline` value to Button.Link.**

Extend Button.Link's existing size axis: `xs | default | sm | lg | inline`. The `inline` size has:
- height: inherit
- typography: inherit
- padding: 0
- gap: 0
- iconSize: inherit

Cheaper to ship (no new component). But conceptually muddies "Button" — buttons are not inherited-typography text fragments.

### Bundle / extractor impact

Optional spec extension to prevent the same miss with future systems:

Add `usageContext` to component manifests:

```json
"usageContext": {
  "variant=Link": {
    "intendedUse": ["standalone-cta", "button-row"],
    "notIntendedFor": ["inline-anchor-in-prose"],
    "alternative": "InlineLink (if available)"
  }
}
```

Consumers (Claude Design) MAY read this and refuse to inline. SP candidate for `BUNDLE_SPEC.md` v0.3.0 if pattern repeats with other DSs.

---

## Finding 2 (MEDIUM IMPACT) — Icon size width is hardcoded

### Symptom

`Button` size=icon (and likely icon-xs / icon-sm / icon-lg by analogy) emits a raw `width: 44px` on the container instead of binding to the `width/w-11` variable.

### Evidence

Cell `46:189` (Default/default/icon) walked during PoC Step 3:

```
className="… h-[var(--height/h-11,44px)] … w-[44px] …"
```

Height binds to `--height/h-11`. Width does not bind — it's a raw px value.

The token `width/w-11 = 44px` exists in Apollo v2's variables. The variable is just not applied to this node.

### Impact

- **Dev handoff:** developers consuming the spec via tokens see height as a variable, width as a literal. Half-bound. Anything that tweaks the height token will leave width stale.
- **Audit signal:** if `width/w-11` later changes from 44 → 46 to comply with a new touch-target standard, every icon Button would break (or rather, fail to update).
- **Inconsistency:** sets a precedent that some properties of the same DS-managed component can be hardcoded. Erodes the discipline.

### Recommended fix

Bind `width` on all four icon sizes to the corresponding `width/w-*` variable:

| Size | Bind width to |
|---|---|
| icon | `{width.w-11}` (44px) |
| icon-xs | hardcoded 36px → needs new token `{width.w-9}` or bind to `{height.h-9}` |
| icon-sm | hardcoded 40px → needs new token `{width.w-10}` or bind to `{height.h-10}` |
| icon-lg | hardcoded 48px → needs new token `{width.w-12}` or bind to `{height.h-12}` |

Apollo v2 currently has `width/w-3, w-4, w-6, w-11` but not w-9/w-10/w-12. Adding them would clean this up. Alternatively reuse height tokens (height and width are equal for square icon buttons).

---

## Finding 3 (LOW IMPACT) — Link variant Figma documentationLink has wrong fragment

### Symptom

`Button` variant=Link cells reference the wrong shadcn/ui doc URL.

### Evidence

Cell `60:244` (Link/default/Default) — its Figma `documentationLink` reads:

```
https://ui.shadcn.com/docs/components/button#ghost
```

It should be:

```
https://ui.shadcn.com/docs/components/button#link
```

### Impact

- Designers + developers clicking the link from Figma land on shadcn's Ghost variant docs, then have to navigate manually.
- AI consumers (anything that reads `documentationLinks` to seed context) get wrong reference material for Link semantics.

### Recommended fix

In Figma, edit `Button` variant=Link `documentationLink` from `#ghost` → `#link`. Trivial. Apply to every Link cell (24 cells share this prop via variant inheritance, but Figma usually allows editing at the variant level once).

### Bundle / extractor benefit (already implemented)

`BUNDLE_SPEC.md` v0.2.0 SP-6 added `codeConnect.quality.perVariantUrlVerified` + `verificationFailures[]`. The PoC bundle's `Button.component.json` already records this failure with `severity: "low"` and `auditFinding` text. A v3 Hardcore audit run will surface this finding automatically going forward.

---

## Summary

| # | Severity | Finding | Source-DS action |
|---|---|---|---|
| 1 | HIGH | Button.Link unusable inline (18px semibold vs body 16px regular) | Add `InlineLink` component (preferred) OR add `size=inline` to Button.Link |
| 2 | MEDIUM | Icon-size widths hardcoded (`44px` etc.) instead of bound to `width/w-*` tokens | Bind icon-size widths to existing or new `width/w-*` tokens |
| 3 | LOW | Link variant documentationLink points to `#ghost` not `#link` | Edit URL fragment in Figma |

All three are real issues in the source DS — not bundle or extractor bugs. The PoC working as designed surfaced them. Ship the fixes when bandwidth allows (post-OOO).

---

## Why this memo exists

The Button PoC's reverse-render smoke test in Claude Design rendered 5 of 6 standard inline contexts correctly. The one failure — the inline-in-sentence Link case — could have been blamed on Claude Design or on the extractor. It was neither. The fault is upstream in Apollo v2's design.

This is the kind of finding ds-architect v2's Hardcore Mode audit is built to surface. v3 extraction makes it inevitable: if a DS has source-side gaps, the round-trip will expose them at render time. The audit feedback loop is part of the value, not a side effect.

Logging these now so they're not lost between the PoC session and the production audit pass after Sep 1.
