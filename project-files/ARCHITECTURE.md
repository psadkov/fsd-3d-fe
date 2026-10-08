# FSD-3D — Frontend Architecture Reference

The *how*. Read [FSD-3D.md](./FSD-3D.md) for the *why*.

**The method is framework-agnostic.** Concepts (the three D's, layers, registry, prototype workflow, testing posture, legacy adoption) hold for any component-based frontend. Framework-specific syntax is confined to §12 (Framework notes), with Vue 3 as primary and React as an equal.

**The document set** (what ships, and lives in the repo): **ARCHITECTURE.md** (this — the method) · **FSD-3D.md** (the why) · **SKILL.md** (the AI's operating rules) · **AGENTS.md** (the same rules for non-Claude tools) · **COMPONENTS.md** (the live registry) · **HISTORY.md** (the session log) · the **README `## FSD-3D` block** (per-project setup). The three D's in one line: **Decompose** = *where* things go (structure), **Delineate** = *how well-shaped* each piece is (no spaghetti), **Deduplicate** = *not building the same thing twice* (reuse).

---

## 1. Layers (unchanged from FSD)

Import rule: **a module imports only from layers strictly below it, never from a sibling slice on its own layer.**

```
app/       providers, router, global styles, entrypoint       (segments only)
pages/     route-level composition of widgets                 (slices → segments)
widgets/   self-contained page blocks — THIN composition      (slices → segments)
features/  user actions that change state (verbs)             (slices → segments)
entities/  business nouns: model, api, plain visual form      (slices → segments)
shared/    domain-agnostic primitives & utils                 (segments only)
```

**Segments** inside a slice: `ui`, `model` (stores/composables/hooks/schemas/logic), `api` (endpoint calls, DTOs, mappers), `lib` (slice-local helpers), `config`. Name segments by **purpose, not essence** — never `components/`, `hooks/`, `composables/`, `types/`.

---

## 2. Decompose — the order and structure (where things go)

Decompose is about **placement**: which layer each piece lives in and how the whole is arranged. On a **greenfield** project it's effectively automatic — you build, and the AI tracks what already exists and splits things into place as it goes; you don't manage it. On an **existing** repo it's a deliberate, opportunistic journey (§9).

Run any piece top-down, stop at the first yes.

| Ask about the piece… | If yes → | Example |
|---|---|---|
| Knows **nothing** about the business? Pure primitive/util? | `shared` | `BaseDropdown`, `Button`, `formatMoney`, `useDebounce` |
| Represents **one business noun** (data / api / plain visual)? | `entities/<noun>` | `entities/game` → `Game`, `getGames`, `GameCard` |
| A **state-changing user action** (a verb)? | `features/<verb>` | `features/add-to-favorites`, `features/place-bet` |
| A **self-contained page block** from features + entities? | `widgets/<block>` | `widgets/game-grid`, `widgets/header` |
| A **whole route** wiring widgets? | `pages/<page>` | `pages/lobby` |

Disambiguators: **noun → entity, verb → feature.** "Would a different app want this, unchanged and business-free?" → `shared`. Carries a business concept but reusable → `entity`. App-specific composition → `widget`/`page`.

**The basement rule.** Low-level UI primitives descend to `shared/ui` on *first write*, not after a second use. You should not see raw markup or ad-hoc logic in widgets/pages — a custom dropdown, modal, field, or tag goes to `shared` immediately (or an `entity` if inherently domain-flavored). Higher layers compose named units; they don't hand-roll primitives.

**Promotion thresholds (so Decompose ≠ over-abstraction):**
- Descend clear **primitives** immediately (by kind).
- Promote the ambiguous **middle** by count — move to `shared`/`entity` on a *second real consumer* (rule of two).
- Genuinely one-off, business-specific code may stay local — but small, single-purpose, behind a public API, so later descent is cheap. *Local is fine; stuck-and-fat is the failure.*

---

## 3. Deduplicate — the registry (COMPONENTS.md) and naming

**COMPONENTS.md** is a single, always-current, **human-editable** catalogue of every reusable component and shared utility. It is the basement for AI analysis: before anything new is built, it's searched, near-duplicates are flagged, and the answer is usually "extend the existing one with a variant." Developers may edit it freely; the AI treats their edits as authoritative and keeps it current each session.

Each entry records:

| Field | Meaning |
|---|---|
| `name` | one canonical, stable code name (e.g. `GameCard`) |
| `aliases` | human names people use ("game tile", "lobby card") |
| `purpose` | one line: what it is |
| `supports` | features it handles (badges, jackpot, compact…) |
| `variants` | named variants |
| `path` | where it lives |
| `origin` | `fsd` or `legacy` — only meaningful on a migrating repo (§9) |
| `formerlyNamed` | old names, so they still resolve after a rename |

**Reuse-first / AI dedup.** New request → match against `purpose` + `supports`. A site's five near-identical cards collapse to **one `GameCard` with variants**; a new "featured" request adds a variant, not a component — and the AI, reading COMPONENTS.md, proposes exactly that.

**Naming truth (name drift).** A team builds a "row of controls" → code names it `ControlRow` / `.control-ui`; it grows meta + statuses and people start calling it "the status bar"; later something else. COMPONENTS.md holds the one canonical `name` plus `aliases`, so "status bar", "controls", and `ControlRow` all resolve to the same unit. When a role genuinely changes: rename the canonical name (cheap — the public API hides internals; §4) and push the old one to `formerlyNamed`; or, if not renaming yet, add the new human term to `aliases`. One lookup, no accidental re-invention.

### Optional in-component metadata (JSDoc)
COMPONENTS.md is authoritative and maintained by hand. As a convenience, component-level metadata may *also* be mirrored into the code as JSDoc — **on by default, deniable per project** (see the README block, §7). It reads as documentation, not data, and stays minimal (name comes from the framework-native field; props from the signature):

```ts
/**
 * @component GameCard
 * @aliases game tile, lobby card
 * @purpose Displays one game: art, title, optional badges/stats.
 * @supports live badge, jackpot amount, provider logo, compact
 * @variants default, compact, featured
 */
```
If a developer strips these tags, the AI leaves them stripped and keeps COMPONENTS.md updated regardless. (Trade-off: because COMPONENTS.md is hand-kept, edits made outside our sessions can drift; the optional backstop is a CI/pre-commit check flagging components missing from it — later, not now.)

---

## 4. Delineate — a strong, single shape for every piece (no spaghetti)

Delineate is about the **quality** of each unit once it's placed: small, single-purpose, one job per file. No god-files — no 300-line component that is secretly also a service and a store. Concretely:

- **One responsibility per file**; colocate the test. Many small cohesive files > few large ones.
- **Separate concerns by segment:** UI in `ui`, state/logic in `model`, requests in `api`. A component that fetches, formats, holds state, and renders is four files, not one.
- A file past ~one screenful is a smell — split it, or part of it wants to descend.

**Clean public API — the supporting tactic.** Each slice (and each segment in `app`/`shared`) exposes one entry point, `index.ts`; outsiders import the slice root, never internals. That's what lets you rename/split/move anything *inside* a slice without breaking a consumer — a well-shaped piece with a clear surface is safe to change. Enforce with a boundary linter (`eslint-plugin-boundaries`, or FSD's Steiger).

```ts
// entities/game/index.ts  — the slice's one front door
export { default as GameCard } from './ui/game-card.vue';
export { useGames } from './model/use-games';
export type { Game } from './model/types';
// ./api/dto, ./model/mappers stay private
```

---

## 5. The prototype → refract workflow

1. **Prototype (raw allowed).** Build the look in one place — raw markup, minimal structure — to validate fast. Keep it isolated (a page/widget or scratch route).
2. **Graduation trigger + mandatory stop.** When the prototype is **approved**, *or* churn crosses a threshold (~3rd round of significant edits, or file > ~250 lines), **stop and announce**: *"This has outgrown a prototype; pausing to refract it into components before adding more."*
3. **Refract.** Apply the three D's; register the new units; the widget/page becomes thin composition.
4. **Build on** the registered pieces.

---

## 6. Testing

(File shape lives under Delineate, §4.) **Behavior through real collaborators. Don't overmock.** If the project chose `write tests = none` at setup, skip this section and the coverage floor. Otherwise, two tiers today:

- **Pure unit** — `model`/`lib` logic (composables, stores, schemas, selectors) called with real inputs/outputs. No network, no MSW. Aim high here.
- **Component / integration** — render a component with its **real** children and store; assert user-visible behavior (Testing Library, query by role/text, `userEvent`). Where the network is involved, intercept it **at the wire with MSW** — the one sanctioned fake. Your components, composables/hooks, and API layer all run for real; only the HTTP response is supplied.
- **E2e** — on the roadmap, **not in scope yet** (see §13). Don't structure it.

**Don't overmock:** never `vi.mock()`/`jest.mock()` your own modules (that's a decompose signal); never stub third-party libs. Besides MSW, fake only clock/randomness/crypto.

**Coverage:** enforced floor **65% overall**. Posture per layer: `model`/`lib` ~90%+; `entities/ui` · `features` · `widgets` ~60–70% (smoke + key interactions); `pages`/`app` thin integration on critical flows. High coverage falls out of behavior tests because logic already lives in small pure files.

---

## 7. Project setup — README block + questionnaire

Setup lives as a **plain-prose block in the project README** (no bespoke config file), under an `## FSD-3D` heading. If it's missing, the skill runs a one-time questionnaire and **writes the answers into the README**. On an **existing project it analyzes the app first** to infer answers, then only asks about genuine gaps — and this analyze-first behavior applies to everything it sets up, not just the questionnaire.

**Where the docs live.** The doc set (`ARCHITECTURE.md`, `FSD-3D.md`, `COMPONENTS.md`, `HISTORY.md`, `AGENTS.md`) sits at the **frontend project's root** — the folder holding the frontend's `package.json`/`src`. In a monorepo that's the frontend package dir (e.g. `apps/web/`, `frontend/`), not the repo root; if the frontend is the whole repo, it's the repo root. The `## FSD-3D` README block goes in that same frontend folder's README.

**Questionnaire** (the normal project-setup decisions; extend as needed):
project type (greenfield / existing) · framework (Vue 3 / React / other) · build tool (Vite / Nuxt / Next / …) · language (TS default) · state (Pinia / Vuex / Redux / Zustand / none) · styling (scoped SFC / CSS Modules / Tailwind / CSS-in-JS) · routing · package manager · repo layout (single / monorepo) · **write tests? (none / unit / unit+component-integration** — e2e→roadmap) · test location (colocated / separate mirror) · in-component JSDoc metadata (allow=default / deny) · **[existing repo only]** new-structure location (`src/` directly / subfolder like `src/fsd/`).

**Setup is a blocking gate.** On a greenfield project the skill must **ask the questionnaire and wait** before scaffolding; on an existing project it infers, then shows what it inferred and asks the user to confirm/fill gaps. It never silently assumes a stack. If `write tests = none`, the coverage floor is skipped and no tests are scaffolded.

**README block template** (declarative prose, not `KEY=VALUE`):

````md
## FSD-3D

This project follows **FSD-3D** (Feature-Sliced Design + Decompose · Deduplicate · Delineate).
See ARCHITECTURE.md for the method.

**Stack**
- Vue 3 + Vite, TypeScript.
- State: Pinia. Styling: scoped SFC styles. Routing: vue-router.
- Single app (not a monorepo). Package manager: pnpm.

**Conventions**
- Tests: unit + component/integration (Vitest + Testing Library + MSW at the network boundary).
  E2e is out of scope for now — not yet designed.
- Test files live next to the component they cover.
- In-component JSDoc metadata: on — generated by default, safe to remove per-file.
- Coverage floor: 65% overall; logic (model/lib) expected higher.
- New structure lives in: src/ (greenfield) · src/fsd/ (this repo, legacy beside it).

**Living docs** _(both human-editable; kept current each session)_
- COMPONENTS.md — component registry; search before creating, extend where it fits.
- HISTORY.md — session log (YYYY-MM-DD.N): what was asked, decided, changed.

_Last reviewed: 2026-07-02.1_
````

---

## 8. HISTORY.md — the log before the commits

Append-only, newest at the bottom, keyed **`YYYY-MM-DD.N`** (date + per-day index — sorts naturally, groups by day, exposes no clock time). One entry per request↔code iteration:

```md
## 2026-07-02.1 — Split lobby prototype into components
**Request:** turn the approved lobby mockup into real structure.
**Decisions:** GameCard gets compact + featured variants (no new card components).
**Changed:** +entities/game/ui/game-card.vue, +shared/ui/base-dropdown; widgets/game-grid now thin.
**State / next:** filters still raw in the widget — refract next.
```

---

## 9. Adoption on an existing (non-FSD) repo

The method adopts **in place, beside legacy — never convert-on-contact.** A naive "decompose everything now" pass is a giant untested refactor that breaks things; that is explicitly forbidden.

**Phase 0 — analyze, don't touch.** Inventory the repo (framework, build, store, styling, tests, existing structure). Build the initial COMPONENTS.md by scanning existing components and **flag the duplicates** ("these five cards look 90% identical"). Write the README block and AGENTS.md. Zero code moved; the output is a map + a plan for you.

**Where the new structure lives — ask.** New folders go **either directly in `src/`** (beside existing folders) **or in a dedicated subfolder like `src/fsd/`** (name it whatever). The subfolder **may live there permanently, or be moved into `src/` later — optional, never a required milestone.** Legacy stays untouched beside it. **No `@` alias is introduced**; the project's existing import conventions/paths are preserved (the Delineate rule still holds — import a slice's `index.ts`, never internals — just expressed in the project's path style).

**Same-name safety without an alias.** The module system already allows a legacy `Button` and a new `.../Button` to coexist as different files. Disambiguation lives in COMPONENTS.md via the **`origin` (legacy | fsd) + `path`** fields: "Button" resolves to the `fsd` one at its path; the legacy twin is listed `legacy` and flagged as a consolidation candidate. New components take the clean canonical names; legacy twins are recorded and retired as they're absorbed (then removed from COMPONENTS.md) — so no permanent double-naming.

**Refactor only what you touch.** Don't proactively rewrite untouched files. When a task opens a fat legacy component, refract only the parts relevant to the task (extract the primitive into the new tree, register it, leave the file thinner), and report what stayed legacy. Convergence is a byproduct of normal work.

**Tests before moving code.** Legacy flips the greenfield order: write a **characterization test** pinning current behavior (real collaborators + MSW at the boundary) *before* extracting, then refract and prove nothing changed. Too tangled to test without heavy mocking → leave it legacy for now.

**Defaults:** grow `shared`-first (least disruptive; strangler-by-feature is the documented alternative), and on duplicates **report-first, draft-on-request**.

**The agent must NOT:** mass-relocate the tree in one PR; rename repo-wide to match FSD; auto-delete "duplicates" (propose, you approve); or block a bugfix on refactoring its file.

**Fastest wins, in order:** stand up COMPONENTS.md + duplicate report (no risk) → boundary linter scoped to the new code (stops new bleeding) → extract highest-reuse primitives to `shared` behind characterization tests → then features, opportunistically.

---

## 10. Agent-entry files (for any coding tool)

Docs describe the method; an agent follows only what its tool actually loads. Different tools read different files. The imperative rules therefore live in **AGENTS.md** (canonical, agent-facing checklist), **mirrored** to the filenames other tools expect: `CLAUDE.md` (Claude Code), `.cursor/rules` (Cursor), `.github/copilot-instructions.md` (Copilot). Keep them as copies of, or one-line pointers to, AGENTS.md. Docs make the right thing clear; the boundary linter + a COMPONENTS.md-completeness check make the wrong thing fail (the eventual backstop).

---

## 11. PR checklist

- [ ] Searched **COMPONENTS.md**; new component justified, or an existing one extended; registry + aliases updated.
- [ ] Each piece **decomposed** to its lowest cohesive layer; primitives in `shared`; widget/page composition-only.
- [ ] Clean **`index.ts`**; no deep imports; import direction respected.
- [ ] Prototype work **graduated** (not left as a raw blob).
- [ ] Tests: pure-unit for logic, MSW-backed component/integration for wiring; only network/time/randomness faked; ≥65% overall.
- [ ] **[legacy]** only touched files refactored; characterization tests written before extraction; `origin`/`path` set.
- [ ] **HISTORY.md** entry appended (`YYYY-MM-DD.N`).

---

## 12. Framework notes

### Vue 3
`.vue` SFC = one cohesive unit (template + script + scoped style). Reusable logic → composables (`use*.ts`) in `model`/`lib`; Pinia stores in `model`. Canonical name via `defineOptions({ name: 'GameCard' })`. A small `shared/ui` primitive:

```
shared/ui/base-dropdown/
├─ base-dropdown.vue   # component (+ optional JSDoc meta)
└─ index.ts            # public API
```
```vue
<!-- base-dropdown.vue -->
<script setup lang="ts">
defineOptions({ name: 'BaseDropdown' });
defineProps<{ options: { label: string; value: string }[]; modelValue: string | null }>();
defineEmits<{ 'update:modelValue': [value: string] }>();
</script>
<template>
  <select :value="modelValue"
          @change="$emit('update:modelValue', ($event.target as HTMLSelectElement).value)">
    <option v-for="o in options" :key="o.value" :value="o.value">{{ o.label }}</option>
  </select>
</template>
```
```ts
// index.ts
export { default as BaseDropdown } from './base-dropdown.vue';
```
Tests: Vitest + `@testing-library/vue` + MSW.

### React
Same slice shape with `.tsx`; canonical name = function name (or `displayName`); reusable logic → hooks in `model`/`lib`.

```tsx
// base-dropdown.tsx
type Option = { label: string; value: string };
export function BaseDropdown({
  options, value, onChange,
}: { options: Option[]; value: string | null; onChange: (v: string) => void }) {
  return (
    <select value={value ?? ''} onChange={(e) => onChange(e.target.value)}>
      {options.map((o) => <option key={o.value} value={o.value}>{o.label}</option>)}
    </select>
  );
}
```
```ts
// index.ts
export { BaseDropdown } from './base-dropdown';
```
Tests: Vitest/Jest + `@testing-library/react` + MSW.

The `meta`/JSDoc shape is **framework-identical**, so COMPONENTS.md and dedup analysis work the same everywhere.

---

## 13. Roadmap (open questions, not in scope yet)

Stated plainly so the current scope stays honest:

- **COMPONENTS.md harvester** — a script to auto-collect the registry from component metadata. **Today the registry is AI-maintained by hand**; the harvester is not built yet.
- **Enforcement tooling** — the boundary linter config plus a CI/pre-commit check that flags components missing from COMPONENTS.md. Docs make the right thing clear; these make the wrong thing fail.
- **E2e testing** — not designed yet. When added, it becomes a third test tier for critical flows only.
- **Legacy promotion tooling** — the paths-only, tooling-driven step to move a `src/fsd/` staging tree up into `src/` once a repo is fully migrated (optional; a repo may stay staged forever).
