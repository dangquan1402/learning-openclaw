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
| 20 | [What Is Special and What to Improve](notes/24-what-is-special-and-what-to-improve.md) | Distinct strengths of OpenClaw, GoClaw, and DeepAgents, plus practical improvement priorities | ⬜ |
| 21 | [Function Calling and Parallel Actions](notes/26-function-calling-and-parallel-actions.md) | How OpenClaw, GoClaw, DeepAgents, and LangGraph use one input to trigger multiple actions | ⬜ |
| 22 | [Agentic Coding Flywheel Setup (ACFS)](notes/27-agentic-coding-flywheel-setup.md) | Overview of Dicklesworthstone's bootstrap stack, planning workflow, beads, and swarm coordination model | ⬜ |
| 23 | [codex-autorunner (CAR)](notes/28-codex-autorunner.md) | Overview of Git-on-my-level's ticket-driven autorunner, hub model, contextspace, and control-plane design | ⬜ |
| 24 | [ACFS vs CAR: Autorun and Agent-Team Coordination](notes/29-acfs-vs-car-autorun-and-agent-teams.md) | Compare ACFS and CAR as separate tracks for workflow method, durable task control, and multi-agent coordination | ⬜ |
| 25 | [Planning Mode Comparison](notes/30-planning-mode-comparison.md) | Compare lightweight runtime planning, supervisor planning, workflow planning, and graph-defined planning | ⬜ |
| 26 | [Corporate Assistant Architecture Fit](notes/31-corporate-assistant-architecture-fit.md) | When workflow-heavy planning helps a corporate assistant and when a lighter assistant flow is better | ⬜ |
| 27 | [Default Loop vs Delegated Execution](notes/32-default-loop-vs-delegated-execution.md) | When systems stay in the main loop, use direct tools, escalate to subagents, or enter team workflows | ⬜ |
| 28 | [Claude Code Memory and Tool Model](notes/33-claude-code-memory-and-tool-model.md) | Deeper look at CLAUDE.md, auto memory, subagent memory, tool scoping, MCP, skills, and permissions | ⬜ |
| 29 | [Memory Model Comparison](notes/34-memory-model-comparison.md) | Compare OpenClaw's file-first memory, GoClaw's retrieval-backed memory, and DeepAgents' programmable memory architecture | ⬜ |
| 30 | [Plan-to-Spawn Transition](notes/35-plan-to-spawn-transition.md) | How plans turn into real subagent launches: tool calls, delegation actions, and runtime primitives | ⬜ |
| 31 | [Four-System Gap Analysis](notes/36-four-system-gap-analysis.md) | What OpenClaw, GoClaw, DeepAgents, and Claude Code are each missing, and which design elements matter most | ⬜ |
| 32 | [Execution-Mode Selection](notes/37-execution-mode-selection.md) | Compare how systems choose between main loop, direct tools, subagents, and workflow/team mode | ⬜ |
| 33 | [Progress Visibility](notes/38-progress-visibility.md) | Compare todos, task boards, status streaming, and user-visible progress across the systems | ⬜ |
| 34 | [Parent-Child Output Control](notes/39-parent-child-output-control.md) | How systems quarantine worker output and keep the parent responsible for the final response | ⬜ |
| 35 | [Tool and Permission Scoping](notes/40-tool-and-permission-scoping.md) | Compare agent capability scoping versus approval policy across the systems | ⬜ |
| 36 | [When Workflow Mode Is Worth It](notes/41-when-workflow-mode-is-worth-it.md) | Distinguish when a board/workflow layer is valuable and when it is unnecessary overhead | ⬜ |
| 37 | [Evals for Delegation and Orchestration](notes/42-evals-for-delegation-and-orchestration.md) | What to evaluate if you want to trust planning, delegation, memory, and workflow behavior | ⬜ |
| 38 | [Channel-Agnostic HITL Design](notes/43-channel-agnostic-hitl-design.md) | How to model structured human-in-the-loop requests once and render them across terminal and chat channels | ⬜ |
| 39 | [System Message Comparison](notes/44-system-message-comparison.md) | Compare how OpenClaw, GoClaw, DeepAgents, and Claude Code construct and use their system messages | ⬜ |
| 40 | [Prompt vs Memory vs Runtime State](notes/45-prompt-vs-memory-vs-runtime-state.md) | Clarify what belongs in the system prompt, durable memory, and live runtime state | ⬜ |
| 41 | [OpenClaw vs Claude Code Tools](notes/46-openclaw-vs-claude-code-tools.md) | Compare assistant-platform tools versus coding-workflow tools and identify the most important gaps | ⬜ |
| 42 | [OpenClaw vs GoClaw vs Claude Code Tools](notes/47-openclaw-vs-goclaw-vs-claude-code-tools.md) | Compare assistant-platform, workflow-platform, and coding-workflow tool surfaces directly | ⬜ |
| 43 | [OpenClaw Full Architecture](notes/48-openclaw-full-architecture.md) | End-to-end architecture of the gateway, sessions, runtime, memory, tools, and subagents | ⬜ |
| 44 | [GoClaw Full Architecture](notes/49-goclaw-full-architecture.md) | End-to-end architecture of the gateway, policy layers, agent loop, memory, teams, and storage | ⬜ |
| 45 | [DeepAgents Full Architecture](notes/50-deepagents-full-architecture.md) | End-to-end architecture of the base prompt, middleware, LangGraph runtime, memory, and subagents | ⬜ |
| 46 | [Claude Code Full Architecture](notes/51-claude-code-full-architecture.md) | End-to-end architecture of the runtime system payload, tools, memory, permissions, and delegation layers | ⬜ |
| 47 | [MCPorter Investigation](notes/52-mcporter.md) | TypeScript runtime/CLI for calling MCP servers directly, generating typed clients, and packaging single-purpose CLIs | ⬜ |

### Ecosystem Research Sub-topics

| # | Topic | Parent | Notes | Status |
|---|-------|--------|-------|--------|
| 22.1 | [ACFS Core Loop](notes/27a-acfs-core-loop.md) | ACFS | Planning substrate vs execution substrate, beads, `bv`, Agent Mail, and swarm coordination rituals | ⬜ |
| 22.2 | [ACFS Manifest and Bootstrap](notes/27b-acfs-manifest-and-bootstrap.md) | ACFS | Manifest-driven installer design, generated scripts, verification, and opinionated VPS bootstrap architecture | ⬜ |
| 22.3 | [ACFS Task and Agent Management](notes/27c-acfs-task-and-agent-management.md) | ACFS | Centralized bead graph, decentralized worker choice, Agent Mail coordination, shared workspace, and concurrency behavior | ⬜ |
| 23.1 | [CAR Tickets and Control Plane](notes/28a-car-tickets-and-control-plane.md) | CAR | Ticket schema, sequencing, contextspace, hub/repo state, surfaces, and destinations | ⬜ |
| 23.2 | [CAR PMA and Chat Surfaces](notes/28b-car-pma-and-chat-surfaces.md) | CAR | PMA, file inbox/outbox, Telegram collaboration policy, and multi-surface operator workflow | ⬜ |
| 23.3 | [CAR Task and Agent Management](notes/28c-car-task-and-agent-management.md) | CAR | Durable tickets, canonical flow state, dispatch/reply handoff, process registry, and multi-surface control | ⬜ |

### Quick Reference
- [Resources & Links](notes/06-resources.md) — Docs, tutorials, community links
- [Glossary](notes/07-glossary.md) — Key terms and definitions

### Progress
- Track tasks on [GitHub Project](https://github.com/users/dangquan1402/projects/2)

---

**Legend:** ⬜ Not started · 🟡 In progress · ✅ Completed
