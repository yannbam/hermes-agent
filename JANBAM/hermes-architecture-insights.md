# Hermes Agent — Architecture Insights

Session: 2026-04-26

---

## 1. Hermes is an AI Agent *Platform*, not just a CLI

Coding agents (Claude Code, Codex, pi) are single-purpose terminal CLIs.
Hermes uses **one agent core** (`AIAgent` in `run_agent.py`) to serve *all*
entry points: CLI, TUI, Discord, Telegram, Slack, WhatsApp, Signal, cron,
batch processing, ACP editor integration, and Python library usage.

```
       CLI    Discord    Telegram    Cron    ACP (VS Code)
        │        │          │          │         │
        └────────┴──────────┴──────────┴─────────┘
                          │
                    ┌─────▼─────┐
                    │  AIAgent   │  ← one class, one loop
                    │ (run_agent)│
                    └─────┬─────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
        Prompt Builder  Provider   Tool Dispatch
                        Resolution  (47 tools, 19 toolsets)
```

---

## 2. Gateway Agent Lifecycle (e.g. Discord)

When a message arrives via a messaging platform:

```
Platform event → Adapter → GatewayRunner → AIAgent.run_conversation() → Response
```

**Agent caching:** The gateway caches `AIAgent` instances per session.
- First message in a session: **creates** a fresh `AIAgent` (builds system prompt, caches it)
- Subsequent messages: **reuses** the cached instance (preserves prompt caching → ~10x cheaper on Anthropic)

A new agent is only created when the config signature changes (model switch, toolset change, etc.).

---

## 3. Working Directory (cwd) for Messaging Platforms

| Context | Default cwd |
|---------|-------------|
| CLI (`hermes`) | `os.getcwd()` — wherever you launched it |
| Messaging (Discord, etc.) | **Home directory** (`~`) |

The resolution chain (from `gateway/run.py`):

```
terminal.cwd from config.yaml
  → if unset or "." / "auto" / "cwd"
    → MESSAGING_CWD env var (deprecated)
      → Path.home()  ← you end up here
```

To change it, set in `~/.hermes/config.yaml`:

```yaml
terminal:
  cwd: /home/jan/src/hermes-agent
```

`MESSAGING_CWD` and `TERMINAL_CWD` env vars are **deprecated** — use `terminal.cwd` in config.yaml.

---

## 4. System Prompt Layers (what the agent actually sees)

Assembled by `AIAgent._build_system_prompt()` once per session:

```
Layer 1: SOUL.md content (raw, unwrapped — IS the agent's identity)
Layer 2: Tool guidance (auto-injected: memory, skills, session search)
Layer 3: Gateway/user system message (rarely used)
Layer 4: Persistent memory (MEMORY.md + USER.md)
Layer 5: Skills index (if skills tools are active)
Layer 6: Project context files (# Project Context header → AGENTS.md etc.)
Layer 7: Timestamp + session metadata
Layer 8: Platform hint (formatting rules for Discord/Telegram/etc.)
```

---

## 5. Context Files — Discovery Algorithm

### SOUL.md — Global identity (HERMES_HOME only)

- Location: `~/.hermes/SOUL.md`
- **Always loaded**, independent of cwd
- Injected **raw** — no wrapper, no preamble
- Completely replaces the hardcoded `DEFAULT_AGENT_IDENTITY`
- The agent does NOT know the file "SOUL.md" exists — the content *is* its identity

### Project context files — Cwd-scoped, first match wins

| Priority | File(s) | Discovery |
|----------|---------|-----------|
| 1st | `.hermes.md` / `HERMES.md` | Walk upward from cwd → git root |
| 2nd | `AGENTS.md` / `agents.md` | Cwd only, no walk |
| 3rd | `CLAUDE.md` / `claude.md` | Cwd only, no walk |
| 4th | `.cursorrules` | Cwd only, no walk |

**Only ONE project context file is loaded** (Python `or` short-circuit).
They are NOT concatenated.

### The walk algorithm (`.hermes.md` only)

```python
for directory in [cwd, cwd.parent, cwd.parent.parent, ...]:
    if directory / ".hermes.md" exists → return it
    if directory is git root → stop
```

Key behaviors:
- In a git repo: walk stops at git root. Never escapes into parent directories.
- No git repo: walk goes all the way to `/`.
- `AGENTS.md`, `CLAUDE.md` do **no walk** — cwd only.

### Global instruction files — current state

There is **no first-class "global instruction file"** separate from SOUL.md.

| File | Agent sees it as... | Global (cwd-independent)? |
|------|---------------------|---------------------------|
| `SOUL.md` | Core identity ("I am...") | ✅ Always |
| `~/.hermes.md` | Project instructions ("# Project Context") | ⚠️ Only when cwd = ~ |
| `MEMORY.md` | Persistent facts | ✅ Always |
| `USER.md` | User preferences | ✅ Always |

---

## 6. Key Differences from Claude Code / Codex

| Concept | Claude Code / Codex | Hermes |
|---------|---------------------|--------|
| Scope | Single CLI tool | Multi-entry-point platform |
| Sessions | In-memory, lost on exit | SQLite + FTS5, persistent, searchable |
| Context files | `CLAUDE.md` with clear wrapper | SOUL.md = raw identity (no wrapper!), AGENTS.md = project instructions (with wrapper) |
| Multi-user | N/A (single terminal) | Per-user session isolation in shared channels |
| Scheduling | None | Built-in cron with platform delivery |
| Tools | Fixed | 47 tools across 19 configurable toolsets |
| Terminal backends | Local only | 6 backends (local, Docker, SSH, Daytona, Modal, Singularity) |
| Memory | None | Persistent MEMORY.md, USER.md, pluggable memory providers |
| Skills | None | Self-improving skill packages, auto-created from experience |
