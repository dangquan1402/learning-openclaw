# Master/Worker Sequence Diagrams

> **Focus:** Concrete end-to-end flow: master spawns worker -> worker runs -> completion is signaled -> master synthesizes -> user gets final answer.

## Why This Note Exists

The earlier notes explain the architecture. This note turns that into **runtime flow diagrams** so it is easier to reason about what actually happens when one agent delegates to another.

## 1. OpenClaw

OpenClaw uses **sub-agent session + announce handoff**.

```text
User
  -> Main agent session
  -> sessions_spawn / subagent launch
  -> Child sub-agent session starts
  -> Child runs tools / LLM loop
  -> Child finishes
  -> Announce step runs
  -> Announce is routed back to parent/requester session
  -> Main agent rewrites/synthesizes the result
  -> User sees final answer
```

### Important detail

The parent does not learn completion by scanning logs. The runtime hands completion back through the **announce mechanism**.

## 2. GoClaw Self-Clone `spawn`

GoClaw self-clone workers use **spawn + bus/announce queue**.

```text
User
  -> Parent agent
  -> spawn(task=..., mode=async|sync)
  -> SubagentManager creates child task
  -> Child goroutine runs
  -> Child executes LLM/tool loop
  -> Child finishes
  -> Announce item created
  -> Announce queue or direct bus publish
  -> Parent session receives completion payload
  -> Parent reformulates result
  -> User sees final answer
```

### Sync variant

For `mode="sync"`:

```text
Parent calls spawn(sync)
  -> child runs
  -> child returns inline result
  -> parent continues same turn
  -> parent answers user
```

So GoClaw has both:

- **inline completion** for sync workers
- **announce/bus completion** for async workers

## 3. GoClaw Team Workflow

GoClaw team mode adds a **task board + durable task state**.

```text
User
  -> Lead agent
  -> Lead creates tasks on team board
  -> Tasks assigned / claimed by members
  -> Members run in parallel
  -> Members update task progress / comments
  -> Member completes task or reports blocker
  -> Task state changes in PostgreSQL
  -> Team/delegation events emitted
  -> Lead receives result / escalation
  -> Lead synthesizes outputs from one or more members
  -> User sees final answer
```

### Important detail

Here, the master can know completion through **both**:

- result/event delivery
- durable task status in the database

This is the richest control model among the systems studied.

## 4. DeepAgents Sync Subagent

DeepAgents sync subagents use **tool-call return**.

```text
User
  -> Parent agent
  -> task(subagent_type=..., description=...)
  -> Child runnable executes in isolated context
  -> Child finishes with one final message
  -> Middleware wraps final message into ToolMessage
  -> Parent receives tool result
  -> Parent reconciles / synthesizes
  -> User sees final answer
```

### Important detail

The parent only gets the **child's final message**, not the full internal trace.

## 5. DeepAgents Async Subagent

DeepAgents async subagents behave more like **remote background jobs**.

```text
User
  -> Parent agent
  -> start_async_task(...)
  -> Remote LangGraph run starts
  -> Parent stores task_id + metadata in async_tasks state
  -> Parent returns control to user

Later:
User asks for status/result
  -> Parent calls check_async_task / list_async_tasks
  -> LangGraph SDK fetches live run status
  -> If complete, result is fetched
  -> Parent summarizes result
  -> User sees answer
```

### Important detail

This is not push-style announce in the same way as OpenClaw/GoClaw async spawn. It is more **tracked background work with explicit status checks**.

## 6. LangGraph Generic Supervisor/Worker Pattern

LangGraph is the primitive layer, so the sequence depends on your graph design.

Typical pattern:

```text
User input
  -> Supervisor node
  -> Supervisor creates worker branches / subgraphs
  -> Workers run
  -> Workers write state updates
  -> Graph joins or returns to supervisor node
  -> Supervisor merges state
  -> Optional interrupt / human review
  -> Final response node
  -> User sees answer
```

### Important detail

Completion is usually represented as **state transition**, not as a chat-style announce.

## Comparison

| System | Spawn/Delegate Mechanism | How Completion Comes Back | Master Control Style |
|--------|--------------------------|---------------------------|----------------------|
| **OpenClaw** | Sub-agent session | Announce back to parent/requester session | Parent rewrites child result |
| **GoClaw spawn** | Self-clone child task | Sync inline result or async bus/announce | Parent reformulates result |
| **GoClaw teams** | Task board + members | DB state + events + result routing | Lead synthesizes team output |
| **DeepAgents sync** | `task` tool | Tool result (`ToolMessage`) | Parent reconciles final child report |
| **DeepAgents async** | Remote LangGraph run | Explicit status/result checks | Parent decides when to collect and answer |
| **LangGraph** | Custom graph/subgraph | State updates / joins / checkpoints | Supervisor logic defined by graph |

## Bottom Line

There are two broad patterns:

### Push model

The child finishes and the runtime **pushes** completion back to the parent.

- OpenClaw
- GoClaw async spawn
- parts of GoClaw team routing

### Pull or state model

The parent checks or joins on **state**.

- DeepAgents async
- LangGraph
- GoClaw team DB state in some workflows

DeepAgents sync is the simplest model of all:

> the child is "done" because the tool call returned.

## Sources

- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [OpenClaw `subagent-announce.ts`](https://github.com/openclaw/openclaw/blob/main/src/agents/subagent-announce.ts)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
- [GoClaw Agent Teams docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [DeepAgents `async_subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/async_subagents.py)
- [LangGraph README](https://github.com/langchain-ai/langgraph/blob/main/README.md)
