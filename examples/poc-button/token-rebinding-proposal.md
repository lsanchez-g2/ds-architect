# Token Rebinding Proposal — Apollo v2 Button

> **Status:** specification-only proposal · v0.1.0 (DRAFT)
> **Targets:** Apollo v2 (SA) Design System — Figma file `3401ZFUHoboOwA6GGjAEsq`
> **Origin:** `audit-findings-for-source.md` Findings 2 (MEDIUM) + 4 (MEDIUM)
> **Decision (2026-05-15):** add missing width tokens + rebind hardcoded pixel values in Apollo v2 Button. Spec lives here until ratified + applied in Figma by Apollo v2 maintainer post-OOO.

---

## 1. Why this exists

Two related audit findings surfaced during PoC extraction:

- **Finding 2 (MEDIUM)** — Container `width` on icon-only Button sizes is hardcoded raw (`44px`, `36px`, `40px`, `48px`) instead of binding to width tokens. `{width.w-11}` (44px) exists; the rest don't.
- **Finding 4 (MEDIUM)** — `iconSize` (the size of leftIcon / rightIcon / Spinner placeholders) is hardcoded raw across most Button cells (`12px`, `14px`, `16px`, `24px`) instead of binding to width tokens.

Same root cause: source DS hardcodes dimensional values where width tokens should bind. SP-4 `$bindingStatus: hardcoded` flagged this automatically across all walked cells. Without SP-4 the drift would have been invisible until a developer noticed during code review.

Additional pattern surfaced in smoke test #4 (cell-level analysis):

- **Semantically-mismatched bindings** — Apollo v2 sometimes presses **height tokens into width duty** (e.g. `icon-lg` width = `{height.h-12}`; iconSize on `icon` cells → `{height.h-4}` instead of `{width.w-4}`). This is a 4th `$bindingStatus` category candidate ("semantically-mismatched") proposed for BUNDLE_SPEC v0.3.0.

The proposal here closes both findings via two parallel actions: (a) add the missing width tokens, (b) rebind every hardcoded dimension in Button cells.

---

## 2. Token gap analysis

### 2.1 What Apollo v2 has today

| Token | Value | Used where |
|---|---|---|
| `width.w-3`  | 12px | Available but not bound to iconSize on size=xs / icon-xs |
| `width.w-4`  | 16px | Available but not bound to iconSize on default / lg / icon / icon-sm |
| `width.w-6`  | 24px | Available but not bound to iconSize on icon-lg |
| `width.w-11` | 44px | Available but not bound to container width on size=icon |

### 2.2 What's missing

| Proposed token | Value | Needed for |
|---|---|---|
| `width.w-9`    | 36px | icon-xs container width |
| `width.w-10`   | 40px | icon-sm container width |
| `width.w-12`   | 48px | icon-lg container width (currently uses `{height.h-12}` as fallback) |
| `width.w-3-5`  | 14px | iconSize on size=sm — **OPTIONAL**, see §2.4 |

### 2.3 Why parallel width tokens instead of reusing height

icon-only Button sizes have square containers (height = width). Apollo v2 currently exploits this by sometimes binding `width` to a `height/h-*` token (e.g. `icon-lg.width = {height.h-12}`). It works in practice — values match — but:

1. **Semantic mismatch.** A height token describing "h-12 row height" is being used to describe "w-12 container width". Renderers reading the bundle have to know this is intentional rather than a bug.
2. **Drift risk.** If `height.h-12` later changes (e.g. row heights compress in a density-mode), `icon-lg.width` would drift unintentionally.
3. **Audit noise.** SP-4 flags this as `$bindingStatus: partial` (or the proposed `semantically-mismatched`) on every walked cell. Adds avoidable signal in audit reports.

Adding parallel `width.w-9/w-10/w-12` tokens at the same numeric values restores token-system hygiene without changing any rendered value.

### 2.4 The 14px question

`width.w-3-5` (14px) is **optional**. Apollo v2's existing width scale is `w-3` (12), `w-4` (16), `w-6` (24), `w-11` (44) — no fractional step. Adding `w-3-5` at 14px breaks the existing whole-number pattern.

Three options for size=sm iconSize:

| Option | Pros | Cons |
|---|---|---|
| **A:** Add `width.w-3-5` (14px) | Preserves current visual size exactly; explicit token | Breaks scale convention; only 1 cell uses it |
| **B:** Rebind to `{width.w-3}` (12px) | Keeps whole-number scale; aligns with icon-xs | -2px visual change on size=sm icons |
| **C:** Rebind to `{width.w-4}` (16px) | Keeps whole-number scale; aligns with default | +2px visual change on size=sm icons |

**Recommended: B (snap down to 12px).** Reasoning: size=sm already pairs with the smaller text-base typography (16/24); a 12px icon fits the compact rhythm better than 16px. Apollo v2 maintainer's call — none of the three is wrong.

---

## 3. Per-cell rebinding plan

### 3.1 iconSize bindings (Finding 4)

Bind every iconSize emitted by Button cells to a width token:

| Cell pattern | Current iconSize emit | Proposed binding |
|---|---|---|
| variant=*, state=*, size=xs        | `12px` raw                  | `{width.w-3}` |
| variant=*, state=*, size=sm        | `14px` raw                  | `{width.w-3}` (Option B) — OR `{width.w-3-5}` (Option A) |
| variant=*, state=*, size=default   | `16px` partial (some cells already bind to `{height.h-4}`) | `{width.w-4}` (replace height-token uses too) |
| variant=*, state=*, size=lg        | `16px` raw                  | `{width.w-4}` |
| variant=*, state=*, size=icon      | `16px` raw or `{height.h-4}` | `{width.w-4}` |
| variant=*, state=*, size=icon-xs   | `12px` raw                  | `{width.w-3}` |
| variant=*, state=*, size=icon-sm   | `16px` raw or `{height.h-4}` | `{width.w-4}` |
| variant=*, state=*, size=icon-lg   | `24px` raw                  | `{width.w-6}` |

### 3.2 Container width bindings (Finding 2)

Bind every icon-only Button container width to a width token:

| Cell pattern | Current width emit | Proposed binding |
|---|---|---|
| variant=*, state=*, size=icon      | `44px` raw                  | `{width.w-11}` (already exists) |
| variant=*, state=*, size=icon-xs   | `36px` raw                  | `{width.w-9}` (**NEW**) |
| variant=*, state=*, size=icon-sm   | `40px` raw                  | `{width.w-10}` (**NEW**) |
| variant=*, state=*, size=icon-lg   | `{height.h-12}` (cross-axis) | `{width.w-12}` (**NEW**) — replace height-token use |

### 3.3 Cells affected

Each rebinding touches every cell at that size × every variant × every state:

| Size | Variants × States | Cells affected |
|---|---|---|
| xs                 | 6 × 6 | 36 |
| sm                 | 6 × 6 | 36 |
| default            | 6 × 6 | 36 |
| lg                 | 6 × 6 | 36 |
| icon               | 5 × 6 (no Link) | 30 |
| icon-xs            | 5 × 6 | 30 |
| icon-sm            | 5 × 6 | 30 |
| icon-lg            | 5 × 6 | 30 |
| **Total cells touched** | | **264** (full matrix) |

In practice this is a one-time bulk-bind operation in Figma — bind the variable once per cell-template variant, propagate through the variant matrix. Apollo v2's variable system supports this.

---

## 4. New tokens to add to Apollo v2

### 4.1 Required (per the chosen rebinding plan)

```
width.w-9   = 36px
width.w-10  = 40px
width.w-12  = 48px
```

If Option A for size=sm:
```
width.w-3-5 = 14px
```

If Option B (recommended): no additional token.

### 4.2 Token metadata (W3C extended, ready to drop into tokens.json)

```json
"width": {
  "w-9": {
    "$value": "36px",
    "$type": "dimension",
    "$description": "36px width — icon-xs Button container.",
    "$extensions": {
      "tailwind": "w-9",
      "figma": { "name": "width/w-9" },
      "addedIn": "Apollo v2.2 (proposed)"
    }
  },
  "w-10": {
    "$value": "40px",
    "$type": "dimension",
    "$description": "40px width — icon-sm Button container.",
    "$extensions": {
      "tailwind": "w-10",
      "figma": { "name": "width/w-10" },
      "addedIn": "Apollo v2.2 (proposed)"
    }
  },
  "w-12": {
    "$value": "48px",
    "$type": "dimension",
    "$description": "48px width — icon-lg Button container (replaces cross-axis use of height.h-12).",
    "$extensions": {
      "tailwind": "w-12",
      "figma": { "name": "width/w-12" },
      "addedIn": "Apollo v2.2 (proposed)"
    }
  }
}
```

### 4.3 Optional (Option A for size=sm)

```json
"w-3-5": {
  "$value": "14px",
  "$type": "dimension",
  "$description": "14px width — size=sm Button iconSize. Off-step in the whole-number scale; OPTIONAL.",
  "$extensions": {
    "tailwind": "w-3.5",
    "figma": { "name": "width/w-3-5" },
    "addedIn": "Apollo v2.2 (proposed, optional)"
  }
}
```

---

## 5. Figma authoring guidance

### 5.1 Add new width variables

In the Apollo v2 Variables panel, in the `width` collection:

1. Add `w-9` = `36`
2. Add `w-10` = `40`
3. Add `w-12` = `48`
4. (Optional) Add `w-3-5` = `14`

Match existing `width.w-*` token formatting (e.g. set `$description` and any `$extensions.tailwind` shortcut Apollo v2 uses).

### 5.2 Apply container-width bindings (Finding 2)

For each icon-only size, open one representative cell (e.g. `46:189` for `size=icon`), select the container frame, and bind the `Width` constraint to the matching width variable. Figma propagates the binding across the variant set.

| Variant set cells | Bind container `Width` to |
|---|---|
| size=icon (44 cells) | `{width.w-11}` |
| size=icon-xs (30 cells) | `{width.w-9}` |
| size=icon-sm (30 cells) | `{width.w-10}` |
| size=icon-lg (30 cells) | `{width.w-12}` |

**Verify after:** the cell's container Width should show the variable name (not a raw number).

### 5.3 Apply iconSize bindings (Finding 4)

For each Button cell, the IconPlaceholder frame (the wrapper around the Lucide icon) has a `Width` × `Height` constraint. Bind both to the matching width token:

| size | iconSize binding (apply to BOTH Width and Height of IconPlaceholder) |
|---|---|
| xs       | `{width.w-3}` (12px) |
| sm       | `{width.w-3}` (Option B; or `{width.w-3-5}` for Option A) |
| default  | `{width.w-4}` |
| lg       | `{width.w-4}` |
| icon     | `{width.w-4}` |
| icon-xs  | `{width.w-3}` |
| icon-sm  | `{width.w-4}` |
| icon-lg  | `{width.w-6}` |

Some cells already partially bind to `{height.h-4}`. Replace those height-token bindings with the matching `{width.w-*}` per the table — same numeric value, semantically correct axis.

### 5.4 What NOT to do

- ❌ Don't keep height tokens bound to width properties even though values match. The whole point of this proposal is to fix the semantic mismatch.
- ❌ Don't add `w-9`/`w-10`/`w-12` to the height collection too. Width and height collections stay separate; each declares its own scale.
- ❌ Don't change the actual pixel values during the rebind. This is a token-binding hygiene fix, not a visual redesign. If size=sm icon at 14px feels wrong, that's a separate design decision.
- ❌ Don't try to also add w-3-5 unless you've committed to keeping size=sm at 14px specifically. Recommended: take Option B (drop to 12px via `{width.w-3}`) and skip the off-step token.

---

## 6. Bundle-side updates after Figma rebind

Once the rebinding lands in Figma, re-run the extraction pipeline:

1. **Pull updated `tokens.json`** — new width tokens appear under `width/`.
2. **Re-walk Button cells** — `$bindingStatus` flips from `hardcoded` / `partial` → `fully-bound` on every iconSize + icon-* container width.
3. **`Button.component.json` patches:**
   - `bindings.size.icon.width`: `"44px"` → `"{width.w-11}"`
   - `bindings.size.icon-xs.width`: `"36px"` → `"{width.w-9}"`
   - `bindings.size.icon-sm.width`: `"40px"` → `"{width.w-10}"`
   - `bindings.size.icon-lg.width`: `"{height.h-12}"` → `"{width.w-12}"`
   - `bindings.size.xs.iconSize`: `"12px"` → `"{width.w-3}"`
   - `bindings.size.sm.iconSize`: `"14px"` → `"{width.w-3}"` (Option B) OR `"{width.w-3-5}"` (Option A)
   - `bindings.size.default.iconSize`: `"{height.h-4}"` → `"{width.w-4}"`
   - `bindings.size.lg.iconSize`: `"{height.h-4}"` → `"{width.w-4}"`
   - `bindings.size.icon.iconSize`: `"{height.h-4}"` → `"{width.w-4}"`
   - `bindings.size.icon-xs.iconSize`: `"12px"` → `"{width.w-3}"`
   - `bindings.size.icon-sm.iconSize`: `"{height.h-4}"` → `"{width.w-4}"`
   - `bindings.size.icon-lg.iconSize`: `"24px"` → `"{width.w-6}"`
   - All `$widthBindingStatus`: `"hardcoded"` → remove (default = `fully-bound`)
   - All `$iconSizeBindingStatus`: `"hardcoded"` / `"partial"` → remove
   - All `$widthAuditFinding` notes → remove (closed)
   - All `$iconSizeAuditFinding` notes → remove (closed)
4. **`audit-findings-for-source.md`:** mark Findings 2 + 4 as **CLOSED** in the summary table; preserve the body sections for historical record.
5. **`MANIFEST.json`:** bump `bundleVersion` to `0.3.0` (additive: new tokens + cleaned bindings). Update checksums.
6. **GAP-7 entry** in `Button.component.json.$step2_known_gaps` → mark CLOSED with cell-walk evidence.
7. **`step3-sample-findings.md`** + **`MILESTONE.md`** → add a "Findings 2 + 4 closed" section.

---

## 7. Validation plan

Re-test in Claude Design after Figma rebind + bundle re-extract. Run smoke-test prompts targeting cells with previously-hardcoded values:

| Prompt | Expected behavior |
|---|---|
| "Render variant=Default, state=Default, size=icon-sm" | Container width emits as `{width.w-10}` (40px). No `$bindingStatus: hardcoded` flag in audit signals. |
| "Render variant=Default, state=Default, size=icon-lg" | Container width emits as `{width.w-12}` (48px), iconSize as `{width.w-6}` (24px). No `$bindingStatus` flags. |
| "Render variant=Default, state=Default, size=sm" | iconSize emits as `{width.w-3}` (12px, Option B). Audit signal for hardcoded 14px disappears. |
| "Audit signals on variant=Outline, state=Disabled, size=icon" | F2 audit flag REMOVED (was: "hardcoded 44px"); F4 audit flag REMOVED. Only finding 3 (doc URL typo) remains. |

Acceptance gate: zero `$bindingStatus: hardcoded` or `$bindingStatus: partial` (in the width / iconSize axes) across all 264 Button cells.

---

## 8. Open questions

- **Option A vs B for size=sm iconSize?** Recommended Option B. Apollo v2 maintainer decides.
- **Do other Apollo v2 components also exhibit hardcoded width / iconSize?** Likely yes. After Button rebind, extend the audit to remaining components in the atom batch.
- **Should width tokens get a Tailwind shortcut field even though Apollo v2 doesn't use Tailwind directly?** Existing tokens already carry `$extensions.tailwind` so consumers can downstream-emit Tailwind classes. Keep the convention.
- **w-3-5 naming convention.** Apollo v2 uses kebab-with-trailing-modifier (`spacing/0-5`, `stroke-width/stroke-1-5`). `w-3-5` follows the same pattern. Alternative `w-3.5` if Apollo v2 wants to support fractional-step tokens explicitly.

---

## 9. Why this approach was chosen

Three alternatives considered:

1. **Add width tokens + rebind (proposed).** Cleanest long-term; restores token-system hygiene; one-time cost.
2. **Continue using height tokens for width.** Faster (no new tokens) but leaves SP-4 / semantically-mismatched flags firing forever. Renderers have to know the height→width pattern is intentional.
3. **Leave hardcoded as-is, document only.** Cheapest now, accumulating debt. Every future audit re-surfaces the same findings.

Option 1 wins because:

- **One-time cost, permanent benefit.** ~1 hour of Figma work + re-extraction. After that, every audit run shows clean `fully-bound` status.
- **Aligns with Apollo v2's existing token convention.** `width.w-*` tokens already exist for some values; this just fills the gaps.
- **Enables future composability.** If Apollo v2 ships density modes (compact / comfortable) later, parallel width/height token collections give clean mode overrides.
- **Removes audit noise.** Future smoke tests stop firing F2 + F4 + GAP-7 + semantically-mismatched flags on every walked Button cell.

---

## 10. Effort + ownership

- **Owner:** lsanchez-g2 (Apollo v2 maintainer, OOO May 26 → Sep 1)
- **Effort:**
  - Add 3 (or 4) width variables in Figma: ~5 min
  - Rebind container widths on 4 icon-only sizes: ~10 min (one bind per size, propagates to all variants/states)
  - Rebind iconSize on 8 sizes: ~20 min
  - Re-run bundle extraction (`ds-architect` PoC steps 1–3 for Button): ~30 min
  - Patch + re-commit bundle artifacts: ~30 min
  - **Total: ~1.5 hours**
- **Blockers:** none (proposal can land in Figma anytime post-OOO).
- **Resume:** when post-OOO Apollo v2 work picks up, this doc is the implementation brief.

---

## 11. Companion fix — Finding 3 (1-line, no proposal doc)

Not in this proposal's scope but worth tracking alongside since it's the same maintainer touching the same Figma file:

**Finding 3:** Button variant=Link cell `60:244` has `documentationLink` pointing to `https://ui.shadcn.com/docs/components/button#ghost` instead of `https://ui.shadcn.com/docs/components/button#link`.

**Action:** open cell `60:244` in Figma → component description panel → edit the URL fragment `#ghost` → `#link`. Apply to every Link variant cell (or once at the variant level if Figma propagates). **1-line fix, no spec doc needed.**

`Button.component.json.codeConnect.quality.verificationFailures[0]` will clear automatically on next extraction.
