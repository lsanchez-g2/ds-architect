# ds-architect v3.0 — Lossless Extraction Roadmap

> Evolves ds-architect from **audit + thin export** (v2) to **audit + lossless serialization + reverse-render verification** (v3).
>
> Goal: any Figma design system extracted via ds-architect can be uploaded to Claude Design (or another Skill-compatible consumer) and reverse-rendered with 100% semantic fidelity and ≥97% pixel fidelity across every component, variant, state, token, and asset.
>
> Companion docs: `BUNDLE_SPEC.md` (the schema contract), `PoC-PLAN.md` (Button validation step).

---

## 0. North Star

**100 components in Figma = 100 components in Claude Design.**
**Fully-rounded button in Figma = fully-rounded button in Claude Design, every time, every variant.**

Achievable for the semantic layer with the right contract. Pixel layer caps at ~97–99% due to renderer differences (font anti-aliasing, continuous corner smoothing). Documented in `BUNDLE_SPEC.md §13`.

---

## 1. What Changes vs. v2

| Layer | v2 today | v3 target |
|---|---|---|
| Audit | 4 layers (technical, UX, a11y, design POV) | Unchanged — keep |
| Modes | Hardcore / Soft / Spec | Unchanged — keep |
| Token export | W3C `$value` + `$type` only, primitives flattened | W3C extended: `$description`, `$extensions`, alias chains preserved, mode-aware, Figma metadata embedded |
| Component export | `[Component].types.ts` + `.stories.tsx` (code) | `[Component].component.json` (API + bindings map) + `[Component].variants.json` (every cell as full node tree) + types + stories |
| Variant matrix | Implicit in code | Explicit: every cell serialized verbatim |
| Assets | Not extracted | Icons (SVG normalized), images (content-hashed), fonts (manifest), screenshots per variant |
| Composition graph | Not produced | `data/graph.json` cross-referencing components ↔ tokens |
| Prototype | Not captured | `data/prototype.json` with triggers, actions, easing, duration |
| Verification | None | `verification/coverage.json` + `pixel-diff.json` + `binding-diff.json` from reverse-render loop |
| Output packaging | Loose folder | Versioned bundle wrapped as Claude Skill |
| HANDOFF.md | Yes | Yes, but generated FROM the bundle data, not free-form |

v2 audit-and-fix is the **input** to v3 extraction. Order: audit → fix → THEN extract clean state. Extracting drift is extracting bugs.

---

## 2. Phased Delivery

### Phase A — Spec lock (done when this PR merges)
- [x] `BUNDLE_SPEC.md` v0.1.0 drafted
- [x] `PoC-PLAN.md` drafted
- [x] `v3-EXTRACTION-PLAN.md` drafted
- [ ] JSON schemas for each bundle file at `verification/schema/*.json` (deferred to Phase B start)

### Phase B — Button PoC
Run `PoC-PLAN.md` end-to-end against Apollo v2 Button.

**Exit criteria:** all acceptance gates in `PoC-PLAN.md §1` met. Spec bumped to v0.2.0 with patch log.

**Estimated effort:** 10 working days.

### Phase C — Atom batch (5 more atoms)
Parallel extraction of: **Input, Checkbox, Switch, Avatar, Icon**.

Each follows the PoC workflow (Steps 1–9). Goal: stress-test the spec across diverse atom archetypes.

- **Input** — exposes text-range overrides (placeholder text styling), focus/error states.
- **Checkbox** — `aria-checked="mixed"` indeterminate state, custom vector glyphs in indicator.
- **Switch** — micro-motion (thumb slide), prototype interactions, dual-color tracks.
- **Avatar** — image fills, fallback initials, `shape=circle` (validates pill round-trip on a different component).
- **Icon** — pure-vector component, validates SVG export pipeline.

**Exit criteria:**
- All 6 atoms (Button + 5) green per `BUNDLE_SPEC.md §10.4`.
- Spec bumped to v0.3.0 with any additive patches.
- Reusable extractor scripts (or use_figma prompt templates) checked into `tooling/`.

**Estimated effort:** 12 working days (parallel-ish; 3 atoms × 2 engineers × 2 days each + 2 day verifier loop).

### Phase D — Molecule batch (5 molecules)
Parallel extraction of: **FormField (Label+Input+Helper+Error), ListItem, MenuItem, ToastItem, Tab**.

These introduce **nested composition** and **instance overrides**. The spec already accommodates them in §6.4 (INSTANCE node schema), but real-world overrides surface edge cases.

**New verification:** override fidelity. If a FormField overrides the Input's placeholder text and error color, the override MUST survive the round-trip.

**Exit criteria:**
- All 11 components (atoms + molecules) green.
- Spec bumped to v0.4.0.
- Override-fidelity verifier added to `verification/binding-diff.json` schema.

**Estimated effort:** 10 working days.

### Phase E — Organism batch (5 organisms)
Parallel extraction of: **Card, Modal/Dialog, Form, Table, Tabs**.

These introduce **deep nesting**, **multi-level overrides**, and **multi-trigger prototype interactions** (e.g. Modal: ON_CLICK opens, ESC closes, click-outside dismisses).

**New verification:** prototype-fidelity. Bundle's `prototype.json` re-rendered as interactive flow MUST match the source Figma prototype.

**Exit criteria:**
- All 16 components green.
- Spec bumped to v0.5.0.
- Prototype-fidelity verifier added.

**Estimated effort:** 12 working days.

### Phase F — Template + pattern batch
- App shells (header + nav + content + footer).
- Auth layout.
- Dashboard layout.
- Marketing landing pattern.

Templates are organism compositions with slot-based content. The spec already supports this; this phase confirms.

**Exit criteria:** 4 templates green. Spec at v0.6.0 (additive only).

**Estimated effort:** 6 working days.

### Phase G — Full DS sweep
Run extractor against the rest of Apollo v2. Auto-batched per atom/molecule/organism level.

**Exit criteria:**
- Every component in Apollo v2 has a bundle entry.
- Aggregate `verification/coverage.json`: 100% on variant + binding coverage.
- Aggregate pixel SSIM ≥ 0.97 mean.

**Estimated effort:** depends on Apollo v2 size; rough: 2–4 weeks.

### Phase H — SKILL.md refactor (the deferred work from earlier audit)
Now safe to refactor SKILL.md per Garima Agarwal Part 3 guidance: progressive disclosure, `references/` split, YAML frontmatter optimization.

Items G1, G2, G3, G5, G6, G7, G8, G9, G10, G11, G12 from prior audit memo, plus:
- G13 — Component manifest schema (DONE via BUNDLE_SPEC.md).
- G14 — Asset pipeline (DONE via Phases B–G).
- G15 — Dependency graph (DONE via `data/graph.json`).
- G16 — Skill bundle output (DONE via §11 of BUNDLE_SPEC.md).
- G17 — Reverse-render verification (DONE via `verification/`).

**Estimated effort:** 5 working days.

### Phase I — Second-DS validation
Run v3 against a non-Apollo source to prove portability. Candidates:
- shadcn/ui reference Figma.
- A public design system: Polaris, Mantine, Carbon.
- A second G2 brand (Capterra, Software Advice, GetApp).

**Exit criteria:** ≥ 95% of components green on a system the extractor was NOT tuned against.

**Estimated effort:** 5 working days.

---

## 3. Aggregate Timeline

| Phase | Effort (days) | Cumulative |
|---|---|---|
| A — Spec lock | 2 (drafting) + 2 (JSON schemas) = 4 | 4 |
| B — Button PoC | 10 | 14 |
| C — Atom batch | 12 | 26 |
| D — Molecule batch | 10 | 36 |
| E — Organism batch | 12 | 48 |
| F — Template batch | 6 | 54 |
| G — Full DS sweep | 15 (variable) | 69 |
| H — SKILL.md refactor | 5 | 74 |
| I — Second-DS validation | 5 | 79 |
| **Total** | **~79 working days (≈ 16 calendar weeks at 5d/w solo)** |

Parallel pairs (atom batch, molecule batch, organism batch) can compress timeline ~30% with two engineers.

**Key constraint:** author OOO May 26 → Sep 1. Realistic milestones:
- **Pre-OOO (May 15 → May 25, ~7 working days):** complete Phase A + start Phase B (likely Steps 0–4 of PoC).
- **Sep 1 onward:** resume Phase B Steps 5–10, then C/D/E.
- **Target full DS coverage:** Q1 2027 if solo, Q4 2026 if co-driver.

---

## 4. Tooling Requirements

| Tool | Purpose | Notes |
|---|---|---|
| Figma MCP / Plugin API | Extractor backbone | `use_figma` already in scope |
| Node walker | Recursive node serialization | Build as `tooling/extract-node.js`; pure function |
| Schema validator | Validate bundle JSON files against `verification/schema/*.json` | Use `ajv` or equivalent |
| Image differ | Pixel SSIM + perceptual hash | `pixelmatch` + `ssim.js` or `image-ssim` |
| Bundle packager | Wrap bundle as Claude Skill (ZIP with folder as root) | Garima Part 3 packaging rule |
| Reverse-render harness | Drive Claude Design with the bundle | Manual first; automate if API access lands |

`tooling/` checked into the repo. Each tool gets a one-page README in `tooling/<tool>/README.md`.

---

## 5. Risks + Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Figma Plugin API doesn't expose enough via MCP | Medium | High | Identify gaps in Phase A; petition for additional MCP tools OR build a separate Figma plugin |
| Claude Design Skills can't load arbitrary file bundles | Medium | High | Test loadability in Phase B Step 8 before scaling; fall back to inline-data SKILL.md if needed |
| Continuous corner smoothing tanks SSIM | High | Medium | Pre-documented in `BUNDLE_SPEC.md §13.2`; SVG clip-path fallback or accept SSIM ≥ 0.85 |
| Variable expressions (conditional values) unsupported | Medium | Medium | Mark in `$extensions.unsupported`; warn consumer; don't block release |
| Reverse-render output non-deterministic | High | Medium | Triple-run verification; report variance; flag SSIM std-dev > 0.02 |
| Author OOO blocks Phase B mid-run | Certain | Medium | Finish Phase A + PoC Steps 0–4 pre-OOO; Steps 5–10 post-OOO |
| Apollo v2 evolves during extraction → bundle goes stale | Medium | Low | Re-run extractor; spec is incremental; bundles are dated per `MANIFEST.source.lastModified` |
| Second consumer (not Claude Design) needs different schema | Low | Medium | Spec is vendor-neutral; consumer-specific shims live in their loader, not the spec |

---

## 6. Out of Scope (v3)

- **Editing Figma from the bundle.** Bundle is read-only output. Future v4 could explore bundle-as-source-of-truth with reverse-write to Figma; not in v3.
- **Multi-file libraries.** If a design system spans N Figma files, v3 supports N bundles. Single unified bundle for cross-file libraries is a v4 candidate (open question in `BUNDLE_SPEC.md §14`).
- **Non-Figma sources.** Penpot, Sketch, etc. not supported. v3 ties to Figma Plugin API surface.
- **AI-generated components.** Bundle is a faithful snapshot of human-authored Figma content. Generative augmentation is a separate skill.

---

## 7. Success Metrics (when v3 ships)

| Metric | Target |
|---|---|
| Apollo v2 components in bundle | 100% of audited components |
| Variant coverage | 100% per component |
| Binding coverage | 100% |
| Pixel SSIM mean across all variants | ≥ 0.97 |
| Fully-rounded round-trip test (pill variants) | 100% semantic pass, ≥ 95% SSIM |
| Time to extract Apollo v2 (end-to-end) | < 4 hours wall-clock |
| Second-DS portability test | ≥ 95% green on a system not tuned against |
| Bundle byte size (Apollo v2 expected) | < 50 MB compressed |
| Time for Claude Design to consume + render any single component | < 30 seconds in fresh conversation |

---

## 8. Open Questions

- [ ] Where do we want bundles to live long-term? Per-project git repo, central artifact registry, or CDN?
- [ ] How often should bundles be re-extracted? Per release? Per merged Figma change? On schedule?
- [ ] Should ds-architect emit a **diff bundle** for incremental updates (only changed components/tokens) or always full snapshots?
- [ ] Authorization: bundles contain proprietary design data. How do we gate access when sharing with external consumers (vendor models, third-party tools)?
- [ ] Telemetry: do we want anonymized fidelity metrics aggregated across users of the skill, to spot consumer-side regressions?

---

## 9. Decision Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-05-15 | Spec-first, then PoC, then scale | Build the contract, prove it on one component, then scale once contract is locked. |
| 2026-05-15 | Button as PoC component | Most representative atom (variants × sizes × states × shapes × slots × interactions). |
| 2026-05-15 | Apollo v2 as primary source | Author owns; real production system; existing audit baseline. |
| 2026-05-15 | Bundle ships as Claude Skill | Matches Garima Agarwal Part 3 packaging; Claude Design auto-loads via progressive disclosure. |
| 2026-05-15 | Don't refactor SKILL.md until Phase H | Risk: refactor against unproven spec → double rework. |

---

## 10. How This Fits With v2-3MODE-PLAN.md

v2-3MODE-PLAN.md established the **input** layer: how to find issues in a design system.
v3-EXTRACTION-PLAN.md establishes the **output** layer: how to package a clean design system for downstream AI consumption.

Both layers active simultaneously:
- Run a Hardcore audit → fix issues → THEN run v3 extraction → upload to Claude Design.
- Or: run v3 extraction directly if the system is already clean; the bundle's `verification/coverage.json` doubles as a maturity scorecard.

No v2 features removed. v3 strictly additive.
