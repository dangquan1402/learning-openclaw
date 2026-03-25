# Learning OpenClaw

A structured knowledge base for learning [OpenClaw](https://github.com/openclaw/openclaw) — the open-source autonomous AI agent framework.

## Table of Contents

### Overview
- [What is OpenClaw?](notes/00-overview.md) — Introduction, history, and key concepts

### Learning Path

| # | Topic | Notes | Status |
|---|-------|-------|--------|
| 1 | [Setup & Installation](notes/01-setup.md) | Environment setup, dependencies, first run | ⬜ |
| 2 | [Architecture](notes/02-architecture.md) | Gateway, Brain, Memory, Skills, Heartbeat | 🟡 |
| 3 | [Skills Development](notes/03-skills.md) | Creating custom skills, SKILL.md format, ClawHub | 🟡 |
| 4 | [Messaging Integration](notes/04-messaging.md) | Connecting to Telegram, Discord, Slack, etc. | 🟡 |
| 5 | [Advanced Topics](notes/05-advanced.md) | Plugins, multi-agent, deployment, security | 🟡 |

### Deep Dives (Sub-tasks)

| # | Topic | Parent | Notes | Status |
|---|-------|--------|-------|--------|
| 2.1 | [Integration Protocol](notes/02a-integration-protocol.md) | Architecture | Channel adapters, message routing, multi-agent | ⬜ |
| 2.2 | [Memory System](notes/02b-memory-system.md) | Architecture | Two-tier memory, MEMORY.md, daily logs, memory tools | ⬜ |
| 2.3 | [Efficient Memory Loading](notes/02c-memory-loading.md) | Architecture | Semantic search, token budgets, optimization patterns | ⬜ |
| 2.4 | [Novel Tech vs Prior Art](notes/02d-novel-tech.md) | Architecture | What makes OpenClaw different from AutoGPT, LangChain, etc. | ⬜ |
| 2.5 | [Single-User vs Multi-User](notes/02e-user-model.md) | Architecture | User model, memory isolation, access control, log privacy | ⬜ |
| 2.6 | [Concurrency & Scaling](notes/02f-concurrency.md) | Architecture | Lane queue, race conditions, 10-user patterns, K8s/Docker | ⬜ |
| 5.1 | [Browser Control & Social Media](notes/05a-browser-control.md) | Advanced | Playwright, posting to FB/X/YouTube, web automation | ⬜ |

### Ecosystem Research

| # | Topic | Notes | Status |
|---|-------|-------|--------|
| 6 | [GoClaw (Go Rewrite)](notes/08-goclaw.md) | Multi-tenant, PostgreSQL, ~35MB RAM, 5-layer security | ⬜ |
| 6.1 | [GoClaw Channels Setup](notes/08a-goclaw-channels.md) | 7 channels: Telegram, Discord, Slack, Zalo, Feishu, WhatsApp | ⬜ |
| 7 | [Chat-with-Documents Landscape](notes/09-chat-with-docs.md) | Top 15 RAG projects ranked: Dify, Open WebUI, RAGFlow, etc. | ⬜ |
| 7.1 | [RAG Limitations & Build vs Buy](notes/10-rag-limitations.md) | Real failures, tool problems, RAG vs long context, when to build your own | ⬜ |
| 7.2 | [Kotaemon Deep Dive](notes/11-kotaemon.md) | PDF citations, 3 GraphRAG modes, hybrid search, 5 reasoning pipelines | ⬜ |
| 8 | [DeepAgents Investigation](notes/12-deepagents.md) | LangChain's agent harness: planning, filesystem context, subagents, CLI, evals | ⬜ |
| 9 | [LangGraph Investigation](notes/13-langgraph.md) | Low-level stateful agent runtime: checkpoints, interrupts, memory, deployment | ⬜ |
| 10 | [Master Agent Control Comparison](notes/14-master-agent-team-control.md) | How OpenClaw, GoClaw, LangGraph, and DeepAgents handle delegation, parallel workers, and final output control | ⬜ |
| 11 | [GoClaw Team Execution](notes/15-goclaw-team-execution.md) | Self-clone spawn vs team delegation, task board control, lead-side synthesis | ⬜ |
| 12 | [DeepAgents Subagent Architecture](notes/16-deepagents-subagent-architecture.md) | Sync/async subagents, output isolation, backends, and supervisor control | ⬜ |
| 13 | [Role-Based vs Supervisor Executors](notes/17-role-based-vs-supervisor-executors.md) | What changed from specialized agents to supervisor/executor patterns, and which system has the stronger self-clone worker | ⬜ |
| 14 | [Master/Subagent Interaction](notes/18-master-subagent-interaction.md) | How completion is signaled: announce, tool return, graph state, or database-backed task status | ⬜ |
| 15 | [Master/Worker Sequence Diagrams](notes/19-master-worker-sequence-diagrams.md) | End-to-end runtime flows for OpenClaw, GoClaw, DeepAgents, and LangGraph | ⬜ |
| 16 | [Subagent Nesting Depth](notes/20-nesting-depth-by-system.md) | Which systems allow `master -> child -> child` and which prefer flat supervisor/worker designs | ⬜ |
| 17 | [Claude Code Agent Teams](notes/21-claude-code-agent-teams.md) | Claude Code subagents vs agent teams, shared task list, teammate messaging, and comparison to GoClaw/DeepAgents | ⬜ |
| 18 | [Claude Code Todo Tracking](notes/22-claude-code-todo-tracking.md) | TodoWrite lifecycle, streaming progress, and how todo tracking differs from a full team task board | ⬜ |
| 19 | [Should OpenClaw Adopt Todos and a Masterboard?](notes/23-should-openclaw-adopt-todos-and-masterboard.md) | Recommendation: add lightweight todo tracking first, then a shared board only for real team workflows | ⬜ |

### Quick Reference
- [Resources & Links](notes/06-resources.md) — Docs, tutorials, community links
- [Glossary](notes/07-glossary.md) — Key terms and definitions

### Progress
- Track tasks on [GitHub Project](https://github.com/users/dangquan1402/projects/2)

---

**Legend:** ⬜ Not started · 🟡 In progress · ✅ Completed
