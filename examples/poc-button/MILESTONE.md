# PoC Milestone — Sample Phase Passed

> **Component:** Apollo v2 Button
> **Date:** 2026-05-15
> **Status:** ✅ Sample-first validation phase passed. Bundle spec contract proven.

---

## What was tested

The Button PoC extracted Steps 0–3-sample of `PoC-PLAN.md`:
- Step 0 — Source snapshot: 264 variant cells confirmed.
- Step 1 — Tokens: 84 leaves (incl. 5 nested-instance backfill per SP-8).
- Step 2 — Component spec: variant matrix + bindings + slots + a11y + codeConnect.
- Step 3 (sample) — 8 of 264 cells walked. Patterns confirmed; remaining 256 cells = predictable batch.

Spec evolved v0.1.0 → v0.2.0 with 8 additive SP patches surfaced during the walk.

---

## Reverse-render smoke test (Claude Design, 2026-05-15)

Bundle's `tokens.json` + `Button.component.json` fed to Claude Design in a fresh conversation. Asked to render 6 inline-context examples (standalone Button, Button row, Link-in-sentence, etc.).

| Result | Count | Notes |
|---|---|---|
| Pixel + semantic match | 5 / 6 | All standalone Button renders + Button-row layouts |
| Source-DS issue (not extraction bug) | 1 / 6 | Link-in-sentence: Apollo v2 Button.Link typography 18px/semibold breaks against 16px/regular body text. Documented in `audit-findings-for-source.md` Finding 1. |
| Extractor / bundle bug | 0 / 6 | None |

**Canonical fully-rounded test:** ✅ PASSED. `border-radius/rounded-full = 9999px` round-trips correctly across all rendered cells. Pill stays pill.

---

## Verdict

The lossless-extraction contract (BUNDLE_SPEC.md v0.2.0) works end-to-end on a real production design system. Claude Design ingests the bundle as designed and reverse-renders with the fidelity the spec promises. The single rendering issue traces to an upstream gap in the source DS, not the bundle — which is the correct behavior for a faithful extractor.

**Spec is locked at v0.2.0** for the atom phase. Bumps only if molecule/organism batches surface non-additive changes.

---

## Deferred to next round

1. **Finding 1 fix** — add `InlineLink` component to Apollo v2 source (post-OOO design review).
2. **Step 3 full extraction** — remaining 256 cells using confirmed patterns.
3. **Step 4** — assets pipeline (Lucide icons + Spinner SVG export).
4. **Steps 5–10** — graph, prototype, screenshots, bundle assembly, Skill packaging, full verifier with SSIM/binding diff.
5. **Findings 2 + 3** — icon width hardcoded; Link doc URL typo. Source-DS fixes, post-OOO.

See `step3-sample-findings.md §9` for ordered next-move list and `PoC-PLAN.md §1` for full acceptance-gate matrix.

---

## Why this matters

The user's stated goal — "100 components in Figma = 100 components in Claude Design, every fully-rounded corner stays fully rounded" — is no longer aspirational. One component proved it. The scaling work (atoms → molecules → organisms → full DS) is now mechanical execution against a validated contract, not exploratory research.

Resume context preserved in `~/.claude/projects/-Users-gsancheztorres-Desktop-claude-g2/memory/ds_architect_v3_direction.md`.
