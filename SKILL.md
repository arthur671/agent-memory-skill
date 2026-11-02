---
name: memory
description: File-based agent memory (a LESSONS.md plus a tasks/ folder with one log per task). Two modes. REMEMBER mode - use whenever the user explicitly asks to memorize or keep something ("remember", "memorize", "don't forget", "keep in mind", "note this down", "save this for later", "from now on ...", "merk dir", "merke dir", "vergiss nicht") or asks to forget it or to recall what was saved; it writes that item into the memory files. INSTALL mode - use when the user asks to set up, add, install, upgrade or repair the memory system, LESSONS.md, task logs or "agent-memory" for an AI tool (pi, Claude Code, Codex, Gemini CLI, Aider, Cursor, Cline, Windsurf, Copilot, ...); it writes the operating rules into that tool's instruction file and replaces the old setup-memory skill.
---

# Memory

Persistent, file-based memory for an AI agent: a shared `LESSONS.md` for general lessons and one log per task under `tasks/`. The instruction file of each tool carries the operating rules (`rules-template.md` next to this file); this skill adds explicit remembering on request and installs the setup.

Several tools can share one memory folder. Never overwrite what is already there.

Find the **memory folder** in the instruction file's rules block (`## Memory files (folder: ...)`). Without that block, the setup is missing: do INSTALL first, then continue.

---

## REMEMBER mode (runtime)

Trigger: the user's message explicitly asks to memorize something, to forget something, or to recall what is saved. Do this in addition to the normal always-on rules in the instruction file, and do it right away, before continuing the task.

### Remember
1. **Pick the store.**
   - Tied to the current task (a fact, decision, path or constraint that only matters for this job) -> the task log under `Facts`. No log yet: create `tasks/<short-slug>.md` from the log template in the instruction file.
   - General and lasting (a user preference or rule, a working command, a tool path, a pitfall) -> `LESSONS.md`. Put user preferences under a `## Preferences` heading and other items under the existing heading that fits best, or `## General` if none fits. Create a heading only when it is missing.
   - When unsure, use `LESSONS.md`. Do not ask the user which store.
2. **Read the target file first.** If an equivalent line exists, update it instead of adding one. If the new item contradicts an old line, replace the old line.
3. **Write one short, concrete line** with the date: `- <what to remember> (YYYY-MM-DD)`. Keep the user's meaning; strip chatter. Nothing task-specific goes into `LESSONS.md`.
4. **Keep the caps.** `LESSONS.md` stays under 60 lines and a task log under about 120. If the write would exceed them, merge or remove outdated lines first. Never drop a line the user explicitly asked to remember unless a newer one contradicts it.
5. **Never store secrets** (passwords, API keys, tokens, private keys). If asked, say you did not store it and, if useful, store only where it lives (for example "API key is in env var X").
6. **Confirm in one line**: what was saved and where, e.g. `Saved to LESSONS.md > Preferences: <line>`.

### Forget
Find the matching line in `LESSONS.md` and the task logs, delete it, and confirm in one line. If several lines match, delete only the clear match and list the others.

### Recall
For "what did I ask you to remember" or "what do you know about X": read `LESSONS.md` and the relevant task log, and answer from them. Do not answer from chat memory alone.

---

## INSTALL mode (setup and upgrade)

### Inputs (find out, do not ask unless truly unknown)
1. **Target tool**: the AI to set up. Default: the tool you are running in. If the user names another, set up that one.
2. **Memory folder**: default `%USERPROFILE%\agent-memory\` (native Windows) or `~/agent-memory/` elsewhere. These are home-folder placeholders, never a hard-coded user name. Resolve the placeholder on the machine you run on (`$HOME` / `$env:USERPROFILE`) and render the resulting absolute path into the rules, because file tools do not reliably expand `~` or `%USERPROFILE%`. Never copy a resolved path from another machine or another user's rules file. Reuse it if it exists; use another path only if the user names one. The path must be valid on the machine the tool actually runs on: a `C:\...` path is only valid on native Windows, never under Linux or WSL (there, use the WSL home or `/mnt/c/...`), and a `/home/...` path is never valid on native Windows. Check the parent folder exists (`test -d` / `Test-Path`). If a path you were given (by the user, a rules file, or an old instruction file) does not exist on this machine, treat it as wrong: do not install it, use the default and say so in the report.
   All tools on one machine share one folder. If another tool's instruction file already has a rules block, read its `folder:` line and reuse that path.
3. **Autonomy**: the template makes the agent work without asking questions, except for the "Hard stop" section (irreversible or external actions are never done unattended; they are written to "Next" as `BLOCKED: needs approval`). If the user wants an interactive agent, delete the line starting "Work autonomously" and the "Do not ask the user" phrases in the copy you install; keep the Hard stop section.

### Steps
1. **Memory folder.** Create it and its `tasks/` subfolder if missing. Create `LESSONS.md` only if missing, containing `# Lessons`, `## Preferences` and `## General`. Create an empty `CURRENT.md` only if missing (the rules tell the agent to keep the active task slug in it). Never truncate or rewrite an existing `LESSONS.md` or task log. An existing `LESSONS.md` without those headings keeps its content: add the two headings above the existing lines only if it has no headings at all.
2. **Find the target's instruction file** (see table). If the tool's location is not in the table or you are unsure, check its docs or ask the user for the path. Prefer the user-level (global) file over a project file unless the user asks for project scope.
3. **Render the rules.** Read `rules-template.md`. Replace `{{MEMORY_DIR}}` with the absolute memory folder path and `{{SEP}}` with `\` on Windows or `/` elsewhere. Always render from `rules-template.md`, never from another copy of the rules.
   **External rules file** (the user points at a file with memory rules, e.g. on a Desktop or share): never install it verbatim. Its paths belong to the machine it was written on, and its wording may be older or newer than the template. Compare it with `rules-template.md`, merge any real improvements into `rules-template.md` itself (keeping `{{MEMORY_DIR}}` and `{{SEP}}` as placeholders and the BEGIN/END markers and the `# Operating rules (apply to every task)` heading), list the differences you took or left, then render and install from the template. Copy the merged template into every tool's `memory/` skill folder so they stay identical.
4. **Install without ever duplicating.** The template is wrapped in `<!-- BEGIN agent-memory rules ... -->` and `<!-- END agent-memory rules -->` markers. Read the whole instruction file and look for an existing copy, in this order:
   - **Marked block(s):** count the BEGIN markers. One: replace it (BEGIN to END inclusive) with the new block. More than one: keep a single block in the place of the first, delete the others, and say so.
   - **Unmarked copy:** no markers, but the file already contains this setup, recognizable by the heading `# Operating rules (apply to every task)` or by `LESSONS.md` together with a `tasks` log folder and a "Log template" section. Replace that whole section (from its heading to the end of the "Finishing" part, or to the next unrelated top-level heading) with the new marked block. Do not append a second copy. Keep every line outside that section.
   - **Different memory rules** (the file has its own, non-matching memory or logging instructions): do not merge silently. Tell the user what you found and ask whether to replace it or keep it.
   - **File missing:** create it with the rendered block.
   - **None of the above:** append the block after a blank line and leave all other content untouched.
   A tool that needs a different file format (for example Cursor `.mdc` needs frontmatter) gets the same block wrapped in that format. The same rule applies: replace, never add a second copy.
5. **Install the skill once, and remove the old one.** In the target tool's skills folder, put this skill at `memory/`, overwriting an existing `memory/` folder (never `memory-2`). Then delete the old `setup-memory/` folder in that same skills folder if it exists, because this skill replaces it. Delete only a folder named exactly `setup-memory` that contains a `SKILL.md` whose frontmatter says `name: setup-memory`; leave anything else alone. Skip this step for tools without a skills folder, since the rules block already carries the remember behavior.
6. **Verify (mechanically, do not just eyeball).** Read the instruction file back and check all of these; fix any failure before reporting:
   - Exactly one BEGIN marker, exactly one END marker, exactly one `# Operating rules (apply to every task)` heading, exactly one `## Hard stop: irreversible or external actions` heading, no unreplaced `{{...}}`.
   - The text from BEGIN to END inclusive is identical to the rendered template (render it again and compare, e.g. with `diff`). Any difference means the block was edited or came from another source: reinstall it from the template.
   - The path in the `## Memory files (folder: ...)` line exists as a directory on this machine (`test -d` / `Test-Path`), contains `LESSONS.md` and `tasks/`, and uses this OS's style (no `C:\` under Linux/WSL, no backslashes in `tasks{{SEP}}` lines under Linux).
   - If other tools on this machine already have the block, their `folder:` line is the same path.
   - `memory/` exists in the tool's skills folder and is identical to this skill (`diff -r`), and `setup-memory/` does not exist. If the tool can be run non-interactively, ask it where its memory files live and to name its skills; say so if it cannot be run.
7. **Report** in a few lines: memory folder, instruction file written, whether `LESSONS.md` was new or existing, whether the old skill was removed, and any conflict you left alone.

### Where each tool reads instructions

| Tool | User-level file | Project-level file |
|---|---|---|
| pi | `~/.pi/agent/AGENTS.md` | `AGENTS.md` |
| Claude Code | `~/.claude/CLAUDE.md` | `CLAUDE.md` |
| Codex CLI | `~/.codex/AGENTS.md` | `AGENTS.md` |
| Gemini CLI | `~/.gemini/GEMINI.md` | `GEMINI.md` |
| Aider | none: add `read: [<file>]` in `.aider.conf.yml` pointing at a `CONVENTIONS.md` that holds the block | `CONVENTIONS.md` |
| Cline | none | `.clinerules` |
| Cursor | none | `.cursor/rules/agent-memory.mdc` (frontmatter `alwaysApply: true`) |
| Windsurf | global rules in its settings | `.windsurfrules` |
| GitHub Copilot | none | `.github/copilot-instructions.md` |

These locations are from memory and tools change them. If a path does not exist or the tool has a different convention, check its documentation instead of guessing.

### Notes
- The memory folder must be reachable by the tool's file tools. If the tool is sandboxed to the project directory, say so and offer a project-local memory folder instead.
- The block is meant for long, unattended runs. Keep the caps from the rules.
- A hand-edited block is overwritten on the next install (the verify step requires it to match the template). Put lasting changes into `rules-template.md` and re-run INSTALL for every tool, instead of editing one tool's block.
- To remove the setup from a tool, delete the text between the markers (inclusive) and the `memory/` skill folder. Leave the memory folder alone unless the user asks.
