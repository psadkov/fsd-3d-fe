# FSD-3D

**Feature-Sliced Design + three disciplines: Decompose · Deduplicate · Delineate.**

*(Written FSD-3D, FSDDD, or FSD³ — same thing.)*

FSD-3D keeps [Feature-Sliced Design](https://feature-sliced.design)'s layers and one-way import rules exactly as they are, and adds the three disciplines FSD leaves to your judgement — the three that decide whether a codebase stays clean or rots.

**The method is framework-agnostic.** The rules below hold for any component-based frontend. Concrete syntax lives in a separate "framework notes" section of the reference; examples are given in **Vue 3** (primary) and **React** (equal), with room for others.

---

## The problem we're solving

Left to "vibe-coding," a frontend rots in a specific, repeatable way:

- **3,000-line files** — everything for a screen piles into one component.
- **New-over-reuse** — it's easier to write a fresh component than to find the existing one, so near-duplicates multiply (a game site ends up with five "card" components that are 90% identical).
- **Divergent fixes** — because those duplicates are *separate* components, a change lands in one and is silently missed in the other four.
- **Painful refactors** — nothing is safe to move; no cleanup, no restructuring; debt compounds.
- **Unpredictable structure** — plain FSD helps but is inconsistent: sometimes everything lands in `widgets`, sometimes `pages`, sometimes `entities`. Good enough isn't good enough.

We want maintainability, real deduplication, honest coverage, and a structure a human *or an AI* can predict. FSD-3D makes that the default outcome instead of the thing you wish you'd done.

## Why not just "use FSD"?

FSD is right about layers, slices, segments, and one-way imports. But it's deliberately quiet on the hard question — *how small, and how far down?* — and its 2.0 revision steers the other way ("pages-first": keep non-reused code high, push down only once reused). That's a fine default for a **single** app, where premature abstraction is the main risk.

FSD-3D targets the opposite situation: **several apps meant to share a design system and logic.** There, "wait until it's reused" is a chicken-and-egg trap — code that never descends never becomes reusable, so it never gets reused. FSD-3D makes the opposite bet on purpose (decompose first), with guardrails so it doesn't tip into over-abstraction.

---

## The three D's

### 1. Decompose — the order and structure (where things go)
Decompose is about **placement**: which layer each piece lives in and how the whole is arranged. The layers and the placement rules (noun→entity, verb→feature) put every piece at its lowest cohesive home; primitives go to `shared` from the first write; widgets/pages end up as thin composition. On a **greenfield** project this is effectively automatic — you build, and the AI tracks what exists and splits things into place as it goes, so you don't manage it. On an **existing** repo it's a deliberate, opportunistic journey (see Adoption). Decompose answers *where*.

### 2. Deduplicate — reuse before you create (never build the same thing twice)
One flat, always-current, human-editable registry — **COMPONENTS.md** — catalogues every reusable piece: canonical code name, the human aliases people use, what it is, and what it supports. Two jobs:
- **Reuse-first / AI dedup.** Before building, search it; if something close exists, *extend it with a variant* rather than spawn a sixth card. "Nothing fit, here's why" is the only valid reason to create new.
- **Naming truth.** It fixes the drift where the thing you call "the status bar" is `ControlRow` / `.control-ui` in code. COMPONENTS.md maps human ↔ canonical name, so everyone (and the AI) resolves the same component regardless of what it's called this week.

### 3. Delineate — a strong, single shape for every piece (no spaghetti)
Delineate is about the **quality** of each unit once it's placed: small, single-purpose, one job per file. No god-files — no 300-line component that is secretly also a service and a store. A clean public API per slice (`index.ts`, internals private) is one tactic under this, not its headline; the point is that every piece has a shape you can read at a glance and change without fear. Delineate answers *how well-shaped*.

---

## The prototype → refract workflow

No big spec up front, and no over-engineering a look nobody's approved:

1. **Prototype (raw allowed).** To validate look and feel fast, raw markup in one place is fine. Don't decompose yet.
2. **Graduation trigger — and the stop.** The moment the prototype is approved **or** it starts churning (repeated significant edits, or it's ballooning), the rule is to **stop and say so**: *"This has grown past a prototype — pausing feature work to split it into components before we go further."* Silently piling onto a raw blob is the failure mode we're killing.
3. **Refract.** Apply the three D's; register the new pieces; leave the widget thin.
4. **Build on** the registered pieces.

## Testing, briefly

Maximum *meaningful* coverage, minimum mock theatre. Two tiers today: **pure unit** tests for logic (no network) and **component/integration** tests that render real components with the network faked at the wire by **MSW** (Mock Service Worker). Don't overmock: never fake your own modules or stub libraries. Baseline floor **65%** overall, logic far higher, glue lighter. *(E2e is on the roadmap, not in scope yet.)*

## What ships with FSD-3D

The method travels as a small set of docs that live in the repo:

- **ARCHITECTURE.md** — the full method (the how).
- **SKILL.md** — the AI's operating rules.
- **AGENTS.md** — the same rules for non-Claude tools (Codex / Cursor / Copilot), mirrored to their expected filenames.
- **COMPONENTS.md** — the live component registry (reuse + naming). Human-editable, kept current each session.
- **HISTORY.md** — the session log, keyed `YYYY-MM-DD.N` — the commit log before there are commits.
- **README `## FSD-3D` block** — per-project setup and conventions.

## Roadmap (not in scope yet)

- A **COMPONENTS.md harvester** — a script to auto-collect the registry from code. *Today the registry is AI-maintained by hand.*
- **Boundary linter + a COMPONENTS.md-completeness check** in CI/pre-commit — docs guide, tooling guarantees.
- **E2e testing** — not designed yet.
- **Legacy promotion tooling** — moving a `src/fsd/` staging tree up into `src/` when a repo is fully migrated.

## Adoption on an existing (non-FSD) repo

FSD-3D adopts **in place, beside the legacy code — it never converts on contact.** It analyzes first, grows the new structure in a location you choose (in `src/` directly, or a subfolder like `src/fsd/` — which may stay there forever or be promoted later), refactors only what a task touches, and writes characterization tests before extracting anything. Details in the reference.

---

## Who this is for

Teams building **more than one** frontend app that should feel like one family and share real logic. One app with no siblings? FSD 2.0's pages-first may serve you better — take the registry, small-files, and prototype-trigger habits, and go easy on aggressive Decompose until reuse actually matters.

---

## Credits

FSD-3D builds on [Feature-Sliced Design](https://feature-sliced.design) (layers, slices, segments, import rules). It is an independent extension and is not affiliated with or endorsed by the Feature-Sliced Design project.
