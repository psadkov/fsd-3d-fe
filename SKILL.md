---
name: fsd-3d-fe
description: "Build, extend, or refactor Vue/React frontends with FSD-3D (Feature-Sliced Design + Decompose, Deduplicate, Delineate). Use for new apps, components, prototypes, refactors, layer/naming/test choices."
---

# FSD-3D Frontend Architecture

FSD-3D = Feature-Sliced Design + three disciplines. Apply all three on every task. When ambiguous, prefer **smaller units, at lower layers, reusing what exists.** Method is framework-agnostic; default stack Vue 3, React equal.

## Setup — RUN THIS FIRST, before writing any code

Check the frontend project's README for a `## FSD-3D` block. **If it's missing, setup has not run — STOP and do setup before building anything. Do not silently assume a stack.**

1. **Ask the questionnaire and wait for the answer.**
   - **Greenfield:** present the questions below and wait — do not scaffold until answered.
   - **Existing repo:** infer what you can from the codebase, then show the user what you inferred and ask them to confirm or fill gaps. Still wait for confirmation.
2. **Install the docs at the frontend project's root** — the folder that holds the frontend's `package.json`/`src`. In a monorepo that's the frontend package dir (e.g. `apps/web/`, `frontend/`), **not** the repo root; if the frontend *is* the whole repo, that folder is the repo root. Copy `project-files/ARCHITECTURE.md`, `FSD-3D.md`, `COMPONENTS.md`, `HISTORY.md`, and `AGENTS.md` there (reconcile, don't overwrite). Mirror `AGENTS.md` to each tool's entry file: `CLAUDE.md` alongside these docs; `.cursor/rules` and `.github/copilot-instructions.md` where those tools look (usually the repo root).
3. **Add the README block** from `project-files/README.fsd-3d.md` (merge under a `## FSD-3D` heading), filled with the answers.

Questionnaire (extend as needed): project type (greenfield/existing) · framework · build tool · language (TS default) · state manager · styling · routing · package manager · repo layout · **write tests? (none / unit only / unit + component-integration)** — e2e is roadmap, not in scope · test location (colocated / separate mirror) · in-component JSDoc metadata (allow=default / deny) · [existing repo] new-structure location (`src/` directly / subfolder like `src/fsd/`).

If **tests = none**, skip the coverage floor and don't scaffold tests (you may still suggest adding them later). If **unit only**, don't write component/integration or MSW tests.

**For the full method** (placement table, Registry schema, legacy adoption, framework notes), read `project-files/ARCHITECTURE.md` on demand — don't load it unless the task needs that detail.

## The three D's

1. **Decompose** — *the order and structure (where things go).* Layers + placement rules put every piece at its lowest cohesive home; primitives to `shared`; widgets/pages thin. On greenfield this is automatic (you build, the AI tracks + splits); on legacy it's a deliberate journey.
2. **Deduplicate** — *reuse, never build the same thing twice.* Before creating anything, search COMPONENTS.md and reuse/extend it. Keep COMPONENTS.md current.
3. **Delineate** — *a strong single shape per piece (no spaghetti).* Small, single-purpose, one job per file; no god-files mixing component + service + store. One `index.ts` public API per slice is the supporting tactic; outsiders never import internals.

## Layers (import only downward; never a sibling slice)
`app → pages → widgets → features → entities → shared`. `app`/`shared`: segments only. Others: slices → segments. Segments: `ui`, `model` (stores/composables/hooks/schemas/logic), `api`, `lib`, `config`. Never name a segment `components`/`hooks`/`composables`/`types`.

## Placement (top-down, stop at first yes)
1. Business-free primitive/util → `shared`. 2. One business noun → `entities/<noun>`. 3. State-changing action (verb) → `features/<verb>`. 4. Self-contained page block → `widgets/<block>`. 5. Whole route → `pages/<page>`.
Noun→entity, verb→feature. **Basement rule:** low-level primitives (dropdown, modal, field, tag) → `shared/ui` on FIRST write; never hand-roll them in a widget/page. Promote the ambiguous middle by rule of two; descend clear primitives immediately. Don't shatter one-offs into ceremony files.

## Prototyping (explicit phase + mandatory stop)
Raw markup in ONE place is acceptable when prototyping a look — do NOT decompose yet. **STOP and announce** the moment it's approved OR churn crosses a threshold (~3rd round of significant edits, or file > ~250 lines): say *"This has outgrown a prototype — pausing to split it into components before continuing,"* then refract. Never keep piling onto a raw blob.

## Deduplicate in practice (COMPONENTS.md)
- Before creating a component, search COMPONENTS.md; if a close match exists (compare `purpose` + `supports`), **extend it with a variant/prop** and flag the near-duplicate to the user, rather than making a new one.
- Each entry: `name` (one canonical), `aliases` (human names), `purpose`, `supports`, `variants`, `path`, `origin` (fsd|legacy), `formerlyNamed`. COMPONENTS.md is human-editable and authoritative — respect developer edits; keep it current every session.
- **Naming drift:** resolve any human alias ("status bar", "controls") to the canonical `name`. On a role change, rename (safe behind the public API) and push the old name to `formerlyNamed`, or add the new term to `aliases`.
- **JSDoc metadata:** if allowed (default), mirror minimal metadata as JSDoc tags on the component; if denied/stripped, leave it and keep COMPONENTS.md updated regardless.

## Delineate
Every piece is small and single-purpose; no god-files. Keep UI/model/api in their segments. Slice root re-exports only what's needed via `index.ts`; internals private. Assume/enable a boundary linter (`eslint-plugin-boundaries` or Steiger).

## Files & testing
One responsibility per file; colocate tests (unless project chose a separate mirror). Many small files > few large.
Two tiers (when tests are enabled): **pure unit** for `model`/`lib` logic (no network) → aim high; **component/integration** rendering real components + store, network faked at the wire by **MSW**. E2e is **roadmap, not in scope** — don't structure it. Don't overmock: never `vi.mock()` your own modules (decompose signal) or stub libs; besides MSW fake only clock/randomness/crypto. Coverage floor **65% overall** (skipped if tests = none); `model`/`lib` ~90%+.

## HISTORY.md (every iteration)
Append one entry, newest at the bottom, keyed `YYYY-MM-DD.N`:
```
## YYYY-MM-DD.N — <short title>
**Request:** … **Decisions:** … **Changed:** … **State / next:** …
```

## Existing (non-FSD) repo — adopt in place, never convert-on-contact
- **Phase 0: analyze, don't touch.** Inventory the repo; build initial COMPONENTS.md by scanning components and **flag duplicates**; install the doc set + README block. Move no code.
- **Location:** ask whether new folders go in `src/` directly or a subfolder like `src/fsd/`. Legacy stays untouched beside it. **Do NOT introduce an `@` alias**; preserve existing import conventions. The subfolder may stay forever or be promoted later (optional).
- **Same-name safety:** legacy and new components may share a name; disambiguate via COMPONENTS.md `origin`+`path`. New code takes clean canonical names; legacy twins recorded and retired as absorbed.
- **Touch-only:** never rewrite untouched files. When opening a fat legacy file, refract only task-relevant parts, register them, leave it thinner, report what stayed legacy.
- **Tests before moving code:** write a characterization test (real collaborators + MSW) before extracting; too tangled to test without heavy mocking → leave legacy.
- **Defaults:** `shared`-first (strangler-by-feature is the alternative); on duplicates report-first, draft-on-request.
- **Never:** mass-relocate in one PR; rename repo-wide; auto-delete duplicates (propose, user approves); block a bugfix on refactoring its file.

## Definition of done
- [ ] Searched COMPONENTS.md; new component justified or existing extended; registry + aliases updated.
- [ ] Decomposed to lowest cohesive layer; primitives in `shared`; widget/page composition-only.
- [ ] Clean `index.ts`; no deep imports; import direction respected.
- [ ] Prototype graduated (not left raw).
- [ ] Pure-unit for logic + MSW component/integration for wiring; only network/time/randomness faked; ≥65%.
- [ ] [legacy] only touched files refactored; characterization tests before extraction; `origin`/`path` set.
- [ ] HISTORY.md entry appended (`YYYY-MM-DD.N`).
