# AGENTS.md

Operating rules for any AI coding agent working in this repo. This repo follows **FSD-3D** (Feature-Sliced Design + Decompose · Deduplicate · Delineate). Full method: `ARCHITECTURE.md`. Read the `## FSD-3D` block in `README.md` for this project's stack and conventions before you start.

> **Mirror this file** to whatever your tool loads: `CLAUDE.md`, `.cursor/rules`, `.github/copilot-instructions.md`. Keep them as copies of, or one-line pointers to, this file.

## Always
- **Decompose (structure — where):** put every piece at its lowest cohesive layer; primitives to `shared/ui` on **first** write; widgets/pages stay thin composition. On greenfield you handle this automatically (track what exists, split as you go); on legacy it's opportunistic (see below).
- **Deduplicate (reuse — not twice):** **search `COMPONENTS.md` before creating any component.** If a close match exists (compare purpose + supports), **extend it with a variant** and tell the user — do not add a near-duplicate. Update `COMPONENTS.md` (name, aliases, purpose, supports, variants, path) for anything you add/rename/extend.
- **Delineate (shape — no spaghetti):** every piece is small and single-purpose; no god-files (a component that's also a service and a store). Keep UI/model/api in their segments. Import from a slice's `index.ts`, never internals; imports flow downward only (`app→pages→widgets→features→entities→shared`); never import a sibling slice.
- **Names:** resolve human aliases ("status bar", "controls") to the canonical name via `COMPONENTS.md`. On a rename, update it and record the old name under `formerly`.
- **Prototyping:** raw markup in one place is fine *while prototyping a look*. The moment it's approved OR churns (~3rd significant edit round, or > ~250 lines), **STOP and say:** "This has outgrown a prototype — pausing to split it into components," then refract.
- **Tests:** pure-unit for logic (no network); component/integration rendering real components with the network faked **only at the wire via MSW**. E2e is roadmap, not in scope. **Never mock your own modules or stub libraries.** Keep overall coverage ≥ 65%.
- **HISTORY.md:** append one entry per iteration, keyed `YYYY-MM-DD.N`.

## On an existing (non-FSD) repo — adopt in place, never convert-on-contact
- Analyze first; move no code in the analysis pass. Build/extend `COMPONENTS.md` and flag duplicates.
- Put new code where the README says (`src/` or a subfolder like `src/fsd/`), **beside** untouched legacy. **Do not introduce an `@` alias**; use existing import conventions.
- Refactor **only files a task makes you open**; leave the rest legacy and say so.
- Write a **characterization test before extracting** anything from legacy.
- Legacy/new components may share a name — disambiguate via `COMPONENTS.md` `origin` + `path`.

## Never
- Never create a component without checking `COMPONENTS.md` first.
- Never leave logic or raw primitives sitting in a widget/page.
- Never deep-import past a slice's `index.ts`.
- Never `vi.mock()`/`jest.mock()` your own modules, or stub third-party libraries.
- Never mass-relocate, repo-wide rename, or auto-delete "duplicates" on a legacy repo — propose, let the user approve.

## Done means
Searched COMPONENTS.md · decomposed to lowest layer · primitives in shared · clean `index.ts` · prototype graduated · behavior tests with real collaborators (only network/time/randomness faked) ≥65% · [legacy] only touched files refactored, characterization tests written · HISTORY.md updated.
