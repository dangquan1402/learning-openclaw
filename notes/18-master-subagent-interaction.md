# Master/Subagent Interaction: How Completion Is Signaled

> **Focus:** How does a master agent know a worker is done? By reading logs, by polling, by database state, or by runtime events?

## Short Answer

In the systems we studied, the master usually does **not** learn completion by reading logs.

Logs are mostly for debugging.

The real interaction path is usually one of these:

- a **tool result** returned directly to the parent
- a **message/announce** routed back into the parent session
- a **state update** in the runtime graph
- a **database-backed task status** for durable workflows

## 1. OpenClaw

OpenClaw uses a **runtime announce flow**, not log inspection.

### How it works

- a sub-agent runs in its own session
- when it finishes, OpenClaw runs an **announce** step
- that announce is routed back to the requester chat or parent session
- the parent agent receives internal runtime context and rewrites it in normal assistant voice

### What tells the parent the child is done?

The completion signal is the **announce handoff**, plus subagent registry/runtime state around that flow.

Relevant mechanisms visible in the repo:

- `subagent-announce.ts`
- `subagent-announce-queue.ts`
- `subagent-registry-*`
- queue lanes for `subagent`

### Role of logs

Logs help you inspect or debug subagents (`/subagents log`), but they are **not** the primary control mechanism.

## 2. GoClaw

GoClaw uses two related mechanisms depending on the pattern.

### A. Self-clone `spawn`

For self-clone subagents, completion is signaled through the **message bus / announce queue**.

From `subagent_exec.go`:

- the child finishes execution
- GoClaw creates an announce item
- it either enqueues that item into the announce queue or publishes it directly through the message bus
- the announce is routed through the **parent session** so the parent can reformulate the result for the user

So for plain `spawn`, the parent knows through an **internal event/message flow**, not by checking logs.

### B. Team workflows

For teams, GoClaw adds **database-backed task state** on top:

- tasks live in PostgreSQL
- statuses move through `pending`, `in_progress`, `completed`, `failed`, etc.
- delegation and task events are also emitted over WebSocket

So GoClaw has both:

- **message-bus completion handoff** for child result delivery
- **database-backed lifecycle state** for durable team coordination

This is the most workflow-heavy model of the group.

## 3. DeepAgents

DeepAgents has two very different paths.

### A. Sync subagents

Sync subagents return a **tool result** back to the parent.

The key flow in `subagents.py` is:

- the parent calls `task(...)`
- the child runnable executes
- the middleware takes the child's final message
- it wraps that final message into a `ToolMessage`
- the parent continues with that result in its own thread

So the parent knows the child is done because the **tool call resolves**.

No database is required for that path.

### B. Async subagents

Async subagents are different:

- they run on remote LangGraph servers
- the parent stores tracked task metadata in `async_tasks` state
- when needed, the parent calls tools like `check_async_task` or `list_async_tasks`
- those tools query the remote run status through the LangGraph SDK (`runs.get`, `threads.get`)

So the parent knows completion through **runtime state + remote run-status checks**, not logs.

## 4. LangGraph

LangGraph is the lowest-level version of the same idea.

It is built around:

- graph state
- checkpoints
- interrupts
- resumable execution

So if you build a master/worker system in LangGraph, the parent usually learns child completion through:

- a returned state update
- a terminal node transition
- checkpointed graph state
- optional human interrupt/review points

LangGraph is about **stateful workflow control**, not log-based coordination.

## Comparison

| System | Main Completion Signal | Are Logs Primary? | Is DB Part of Control Flow? |
|--------|------------------------|-------------------|-----------------------------|
| **OpenClaw** | Announce routed back to parent/requester session | No | Not primarily |
| **GoClaw** | Bus/announce for child results; task state for teams | No | Yes, for team/task workflows |
| **DeepAgents** | Sync: tool result. Async: tracked state + remote run status | No | Not required locally; remote LangGraph server may persist runs |
| **LangGraph** | Graph state / checkpoints / node completion | No | Optional depending on checkpointer/store backend |

## Bottom Line

The clean mental model is:

- **logs** = observability
- **bus/tool/state/db** = coordination

So when the master "knows" a subagent is done, it usually does not mean:

> "read the logs and infer completion"

It usually means:

> "the runtime delivered a completion signal through a message, tool return, state update, or task status transition"

## Sources

- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [OpenClaw Queue docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/queue.md)
- [OpenClaw `subagent-announce.ts`](https://github.com/openclaw/openclaw/blob/main/src/agents/subagent-announce.ts)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
- [GoClaw Agent Teams docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [DeepAgents `async_subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/async_subagents.py)
- [LangGraph README](https://github.com/langchain-ai/langgraph/blob/main/README.md)
