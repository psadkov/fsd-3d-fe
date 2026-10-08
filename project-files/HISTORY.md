# HISTORY

Append-only log of request ↔ work iterations. Newest at the bottom. Keyed `YYYY-MM-DD.N` (date + per-day index): sorts naturally, groups by day, exposes no clock time. The commit log before there are commits; keeps AI and humans in sync across sessions.

Entry format:
```
## YYYY-MM-DD.N — <short title>
**Request:** what was asked
**Decisions:** choices made
**Changed:** files / structure touched
**State / next:** where things stand, what's next
```

---
