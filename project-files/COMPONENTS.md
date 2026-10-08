# COMPONENTS.md

The component registry — the source of truth for **reuse** and **naming**. Search this before creating any component; if something close exists, extend it with a variant instead of making a new one. **Human-editable:** edit names, aliases, and descriptions freely; the AI keeps it current and respects your edits.

Each entry: **name** (canonical, stable) · **aliases** (what people call it) · **purpose** · **supports** · **variants** · **path** · **origin** (`fsd`|`legacy`, only relevant while migrating) · **formerly** (old names, so they still resolve).

> On a greenfield repo, ignore the `origin` column. On a legacy migration, `fsd` = the new tree, `legacy` = pre-existing twin to be consolidated and then removed from this list.

---

## shared

### BaseDropdown
- **aliases:** dropdown, select, picker
- **purpose:** Single-select dropdown bound to a value.
- **supports:** option list, controlled value / v-model, disabled
- **variants:** default
- **path:** `shared/ui/base-dropdown`
- **origin:** fsd

_(add: Button, Modal, TextField, Tag, formatMoney, useDebounce, …)_

## entities

### GameCard
- **aliases:** game tile, lobby card, game card
- **purpose:** Displays one game — art, title, and optional badges/stats.
- **supports:** live badge, jackpot amount, provider logo, compact layout
- **variants:** default, compact, featured
- **path:** `entities/game/ui/game-card`
- **origin:** fsd
- **formerly:** GameCardLive, GameCardJackpot, GameCardPromo, GameCardNew, GameCardCompact
- **note:** consolidated five near-identical cards into one with variants. New card requests → add a variant here, do not create a new component.

## features

_(add: add-to-favorites, place-bet, login-by-email, …)_

## widgets

_(add: game-grid, header, …)_

---

### How to use this file
- **Before creating:** scan `purpose` + `supports` for a match. Close match → extend it (new variant/prop) and note it here. No match → create, then add an entry.
- **When naming drifts:** a thing you now call "the status bar" but coded as `ControlRow` stays one entry — add "status bar" to its aliases. If you rename the code, update `name` and add the old one to `formerly`.
- **Legacy twins:** keep both listed with distinct `path`s and `origin`; when the legacy one is absorbed, delete its entry.
