# Architecture

OpenClaw has a **five-component architecture**. Understanding how these work together is key to building on the platform.

## 1. Gateway (Control Plane)

The Gateway is a **single long-lived WebSocket server** that serves as the central hub.

- **Default address:** `ws://127.0.0.1:18789` (localhost-only for security)
- **Always-on** -- runs as a background service

**Responsibilities:**
- Message routing across all messaging channels
- Session management (agent sessions, device pairing)
- Hosting the Control UI and WebChat
- WebSocket API for real-time communication

**Channel Adapters** handle three jobs per platform:
1. **Inbound normalization** -- convert platform payloads to OpenClaw format
2. **Outbound rendering** -- convert responses to platform-specific formats (Slack blocks, Telegram keyboards)
3. **Auth/webhook management** -- verify signatures, refresh tokens

## 2. Brain (Agent Loop)

The Brain is the **LLM-powered reasoning engine** using the ReAct (Reason + Act) cycle.

### The Agent Loop Step-by-Step

```
Message In -> Context Assembly -> LLM Inference -> Tool Calls -> Response Out
                                       ^                |
                                       |________________|
                                      (loop up to 20x)
```

1. **Context Assembly** -- System prompt built from 4 sources:
   - Base prompt (core instructions)
   - Skills prompt (only relevant skills injected)
   - Bootstrap files (`AGENTS.md`, `SOUL.md`, `TOOLS.md`)
   - Per-run overrides

2. **Memory Injection** -- Semantic search for relevant past conversations + today's/yesterday's daily logs

3. **LLM Inference** -- Prompt streamed to configured model provider

4. **Tool Call Interception** -- Model requests a tool call -> runtime executes it (optionally in Docker sandbox) -> result fed back

5. **Loop** -- Model sees tool result, decides to call another tool or produce final reply. **Up to 20 iterations per request.**

6. **Response Streaming** -- Reply flows back through Gateway -> channel adapter -> user's app

**Concurrency:** Runs are serialized per session to prevent races.

**Hook System:** `before_tool_call` / `after_tool_call` hooks for intercepting tool params/results.

## 3. Memory System

Memory is **plain Markdown files on disk**. No hidden state -- the model only remembers what's written to files.

### Two-Tier Architecture

| Tier | Location | Purpose |
|------|----------|---------|
| **Daily Logs** | `memory/YYYY-MM-DD.md` | Append-only daily logs. Today's + yesterday's auto-loaded |
| **Long-term Memory** | `MEMORY.md` (workspace root) | Durable facts, decisions, preferences |

### Default Workspace Layout

```
~/.openclaw/
  ├── memory/
  │   ├── 2026-03-24.md    # Today's daily log
  │   ├── 2026-03-23.md    # Yesterday's daily log
  │   └── ...
  ├── MEMORY.md             # Long-term memory
  ├── preferences.md        # User preferences
  ├── contacts.md           # People the agent knows
  ├── projects.md           # Active projects
  └── learnings.md          # Things figured out
```

### Agent Memory Tools

- `memory_search` -- semantic recall over indexed snippets (vector index over MEMORY.md + memory/*.md)
- `memory_get` -- targeted read of a specific file/line range

### Auto Memory Flush

Before context compaction, OpenClaw triggers a **silent agentic turn** (no user notification) reminding the model to persist important info to memory files. This prevents context loss during long conversations.

### Design Philosophy

- All memories are **human-readable and editable**
- Stored **locally** (not in cloud databases)
- You can directly edit memory files with any text editor

## 4. Skills System

Skills teach the agent how to combine tools to accomplish tasks. Think of them as **textbooks** for the agent.

### SKILL.md Format

```yaml
---
name: my-skill
description: "Manage tasks via Todoist API. Use when: user mentions tasks or todos."
version: 1.0.0
user-invocable: true
disable-model-invocation: false
metadata:
  openclaw:
    primaryEnv: TODOIST_API_KEY
    requires:
      env:
        - TODOIST_API_KEY
      bins:
        - curl
---

# Instructions (Markdown body)
When the user asks to manage tasks...
```

### How Skill Eligibility Works

1. Load bundled + local skills
2. Filter by: binary presence, env vars, config keys, platform
3. Only eligible skills appear in the prompt
4. **Critically:** Only skills relevant to the current turn are fully injected (name + description used for matching first)

### Skill Locations

- User skills: `~/.openclaw/workspace/skills/<skill-name>/SKILL.md`
- Project-local: `.openclaw/skills/<skill-name>/SKILL.md`

### ClawHub Marketplace

- **13,700+** community-built skills
- CLI: `openclaw skill search <query>`, `openclaw skill install <name>`
- Registry: [github.com/openclaw/clawhub](https://github.com/openclaw/clawhub)

> **Security Note:** In Feb 2026, the "ClawHavoc" supply chain attack poisoned ClawHub with 1,184 malicious skills. Always review skills before installation.

## 5. Heartbeat (Scheduler)

The Heartbeat enables **proactive behavior** -- the agent can act without user prompting.

### How It Works

1. Every N minutes (default: 30), Gateway sends agent a heartbeat prompt
2. Agent wakes up, reads `HEARTBEAT.md`, checks for pending tasks
3. Either sends `HEARTBEAT_OK` (silent) or messages the user

### Heartbeat vs Cron

| Feature | Heartbeat | Cron |
|---------|-----------|------|
| **Timing** | Regular interval (30 min default) | Precise cron expressions with timezone |
| **Session** | Main session, full context | Isolated `cron:<jobId>` session |
| **Best for** | Inbox monitoring, calendar awareness, batched checks | Exact-time delivery, standalone tasks |
| **Context** | Full main-session context | Runs in isolation |

**Key advantage:** One heartbeat can batch multiple checks (inbox, calendar, weather, notifications) instead of needing separate cron jobs.

## End-to-End Message Flow

```
1. User sends message on Telegram/Slack/etc.
2. Channel adapter normalizes payload
3. Gateway routes to correct agent session
4. Context assembled (bootstrap files + relevant skills + memory)
5. LLM inference (streamed)
6. ReAct loop: reason -> tool call -> result -> repeat (up to 20x)
7. Final response streamed back
8. Channel adapter renders to platform format
9. Delivered to user
10. Memory persisted to daily log / MEMORY.md
```

## Plugins vs Skills

| Aspect | Skills | Plugins |
|--------|--------|---------|
| **What** | Markdown instructions for the agent | TypeScript modules for the runtime |
| **Level** | Agent/prompt level | Infrastructure level |
| **Language** | Markdown + optional scripts | TypeScript/JavaScript |
| **Can do** | Teach agent new capabilities | Add channels, tools, hooks, CLI commands |
| **Registry** | ClawHub (13,700+) | Fewer, more structural |

**Key distinction:** Plugins are **code** that extends the runtime. Skills are **text** that extends the agent's knowledge. Plugins can contain skills, but not vice versa.
