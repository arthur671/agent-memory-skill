# Agent Memory Skill

Persistent, file-based memory system designed for autonomous AI coding agents (such as **pi**, **Claude Code**, **Gemini CLI / Antigravity**, **Codex**, **Cursor**, **Aider**, etc.).

It solves two fundamental issues with LLM coding agents:
1. **Cross-session amnesia**: Agents forget user preferences, host quirks, and project facts when a conversation ends.
2. **Context compaction loss**: During long sessions, automated context compaction strips out fine-grained negative constraints ("tried approach X with parameter Y, resulted in Z").

---

## Architecture

The system uses plain Markdown files in a shared or local directory (default: `~/agent-memory/` on Unix or `%USERPROFILE%\agent-memory\` on Windows):

- **`LESSONS.md`**: High-value, general lessons and persistent user preferences. Kept concise (capped at ~60 lines) so it can be loaded directly into context.
- **`tasks/<slug>.md`**: Dedicated task logs following a strict format (`Goal`, `Status`, `Facts`, `Tried`, `Stuck`, `Next`). Never retry an approach that already failed twice.
- **`CURRENT.md`**: A single-line pointer tracking the slug of the active task.

---

## Supported Tools & Instruction Files

| Tool | Global / User-Level File | Project-Level File |
|---|---|---|
| **pi** | `~/.pi/agent/AGENTS.md` | `AGENTS.md` |
| **Claude Code** | `~/.claude/CLAUDE.md` | `CLAUDE.md` |
| **Gemini CLI / Antigravity** | `~/.gemini/GEMINI.md` | `GEMINI.md` |
| **Codex CLI** | `~/.codex/AGENTS.md` | `AGENTS.md` |
| **Cursor** | *N/A* | `.cursor/rules/agent-memory.mdc` |
| **Cline** | *N/A* | `.clinerules` |
| **Windsurf** | Global rules in settings | `.windsurfrules` |
| **Aider** | Added via `.aider.conf.yml` | `CONVENTIONS.md` |
| **GitHub Copilot** | *N/A* | `.github/copilot-instructions.md` |

---

## Installation

### Automatic (Prompt an Agent)
Copy this skill directory into your agent's skills directory (e.g. `~/.pi/agent/skills/memory/`, `~/.claude/skills/memory/`, or `~/.gemini/config/skills/memory/`), then ask your agent:

> *"Install the memory skill for yourself."*

The agent reads `SKILL.md`, identifies its environment, creates the memory store, renders `rules-template.md` with resolved paths, and mounts the operating rules into its instruction file without duplicate entries.

### Manual Setup
1. Create your memory directory:
   ```bash
   mkdir -p ~/agent-memory/tasks
   touch ~/agent-memory/CURRENT.md
   cat << 'EOF' > ~/agent-memory/LESSONS.md
   # Lessons

   ## Preferences

   ## General
   EOF
   ```
2. Render `rules-template.md`: replace `{{MEMORY_DIR}}` with the absolute path to your memory directory (e.g. `/home/user/agent-memory/` or `C:\Users\username\agent-memory\`) and `{{SEP}}` with `/` (Unix) or `\` (Windows).
3. Paste the rendered block into your agent's instructions file (e.g., `AGENTS.md`, `CLAUDE.md`, or `GEMINI.md`).

---

## Runtime Usage

Once installed, the operating rules run autonomously in every session. You can also explicitly trigger memory operations:

- **Remember**: *"Remember that on this system ffmpeg needs -c:a copy for mp4 remuxing"*
- **Recall**: *"What did I ask you to remember?"* or *"What do you know about our database setup?"*
- **Forget**: *"Forget the rule about X"*

---

## License

MIT License. See [LICENSE](LICENSE) for details.
