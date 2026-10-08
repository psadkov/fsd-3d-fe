# Installing the FSD-3D skill

A step-by-step guide to installing and using the **fsd-3d-fe** Agent Skill. A "skill" is a folder with a `SKILL.md` that Claude loads automatically when a task matches its description. The same folder works across Claude Code, claude.ai, and the Claude API.

---

## What's in this package

```
fsd-3d-fe/
├─ SKILL.md                # the skill: trigger + operating rules + a setup step
├─ README.md               # this guide
├─ LICENSE                 # MIT
└─ project-files/          # the FSD-3D doc set the skill installs into your repo
   ├─ ARCHITECTURE.md      # the full method (also read on demand)
   ├─ FSD-3D.md            # the why / manifesto
   ├─ COMPONENTS.md        # the component registry (reuse + naming)
   ├─ HISTORY.md           # the session log
   ├─ AGENTS.md            # rules for non-Claude tools (Codex / Cursor / Copilot)
   └─ README.fsd-3d.md     # the README block to merge into your project
```

Only `SKILL.md` is required for a skill to work; everything in `project-files/` is data the skill uses.

---

## Install

Pick the surface you use. You can install in more than one.

### Claude Code

1. Install it from GitHub — https://github.com/psadkov/fsd-3d-fe — into a skills directory:
   - **Global** (available in every project):
     ```bash
     git clone https://github.com/psadkov/fsd-3d-fe ~/.claude/skills/fsd-3d-fe
     ```
   - **Project-only** (shared with the repo), from your repo root:
     ```bash
     git clone https://github.com/psadkov/fsd-3d-fe .claude/skills/fsd-3d-fe
     ```
   Or just ask Claude Code: *"Clone https://github.com/psadkov/fsd-3d-fe into ~/.claude/skills/fsd-3d-fe"*.
2. In an active session, reload so the new skill is picked up (`/reload-plugins`, or restart Claude Code).
3. It now loads automatically when relevant. You can also invoke it by name.

```
~/.claude/skills/
└─ fsd-3d-fe/
   ├─ SKILL.md
   └─ project-files/…
```

### claude.ai (Pro, Team, or Enterprise)

1. Go to **Settings → Capabilities → Skills**.
2. Choose **Upload skill** and select the `fsd-3d-fe.zip` (or the zipped folder).
3. Enable it. Claude uses it automatically when your request matches.

### Claude API / Claude Platform

1. Upload the skill via the **Skills API** (see the Skills API quickstart in the Claude docs).
2. Reference the uploaded skill in your requests.
3. Note: for API skills, any dependencies must be pre-installed in the execution container — this skill ships no scripts or dependencies, so there's nothing extra to install.

---

## First use — what the skill does

The first time you use it in a project, the skill runs its **setup step**:

1. Copies the files from `project-files/` into your **frontend project's root** — the folder that holds the frontend's `package.json`/`src`. In a monorepo that's the frontend package dir (e.g. `apps/web/`, `frontend/`), **not** the repo root; if the frontend is the whole repo, that's the repo root. So `ARCHITECTURE.md`, `FSD-3D.md`, `COMPONENTS.md`, `HISTORY.md`, and `AGENTS.md` become real files there. It **reconciles** rather than overwrites if any already exist.
2. Mirrors `AGENTS.md` to the filenames other tools read (`CLAUDE.md`, `.cursor/rules`, `.github/copilot-instructions.md`).
3. Merges the **README block** (`project-files/README.fsd-3d.md`) into your project's `README.md`.
4. Runs a short **setup questionnaire** (framework, build tool, state, styling, tests, etc.) and writes the answers into that README block. On an existing repo it analyzes the app first and only asks about gaps.

After setup, it reads `project-files/ARCHITECTURE.md` **on demand** only when a task needs the full detail — so it stays lightweight the rest of the time.

---

## Verify it's working

Ask something that should trigger it, for example:

> "Add a game card component to the lobby."

Claude should apply FSD-3D — search `COMPONENTS.md` first, place the piece at the right layer, keep the widget thin, and update the registry and `HISTORY.md`. In Claude Code you can also invoke it explicitly by name.

If it doesn't trigger when you expect, the `description` in `SKILL.md` is what decides invocation — make it a little more explicit about the situations that should fire it.

---

## Troubleshooting — "it didn't ask me any setup questions"

Two common causes:

1. **It isn't actually installed as a skill.** Asking Claude to "install the skill" inside a chat does *not* register it — Claude just reads the files as context. A skill only activates when the folder is in a skills location: `~/.claude/skills/fsd-3d-fe/` (Claude Code) or uploaded in **claude.ai → Settings → Capabilities → Skills**. Put it there, reload, and try again.
2. **It triggered but assumed defaults.** A skill can't force an interactive prompt — Claude follows the instructions if it judges it should. The setup step is now written as a **blocking gate** ("STOP and ask before building"), which makes it far more likely to ask. If it still proceeds, prompt it explicitly:

   > "Run FSD-3D setup first — ask me the questionnaire before writing any code."

You can re-run setup anytime by deleting the `## FSD-3D` block from the frontend README (that's the flag the skill checks) and asking it to set up again.

## Update or remove

- **Update:** `git -C ~/.claude/skills/fsd-3d-fe pull` (Claude Code) or re-upload the zip (claude.ai). Your project's copies of the docs are independent and won't be touched.
- **Remove:** delete the folder from your skills directory, or disable/delete it in claude.ai Settings. Removing the skill does not remove the docs already installed in your repo.

---

## A note on trust

Only install skills from sources you trust — ones you authored or got from Anthropic. A skill can direct Claude to run code or use tools, so review `SKILL.md` and any bundled files before enabling one from an unknown source. This skill contains only Markdown (no scripts).

## License

MIT — see [LICENSE](./LICENSE). FSD-3D builds on [Feature-Sliced Design](https://feature-sliced.design); not affiliated with that project.
