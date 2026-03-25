# Subagent Nesting Depth by System

> **Focus:** Can a sub-agent spawn another sub-agent, or is the architecture limited to `master -> child`?

## Short Answer

These systems do **not** all treat nesting the same way.

Some allow bounded nesting, some discourage it, and some leave it entirely up to the workflow designer.

## 1. OpenClaw

OpenClaw explicitly supports **bounded nesting**.

- default: `maxSpawnDepth = 1`
- result: `main -> sub-agent`
- if configured to `maxSpawnDepth = 2`:
  - `main -> sub-agent -> sub-sub-agent`
- depth 2 is the leaf

So OpenClaw can support an **orchestrator pattern** where a depth-1 child becomes a temporary coordinator and spawns depth-2 workers.

## 2. GoClaw

GoClaw's current `spawn` path is designed as a **self-clone leaf worker**, not as a recursive hierarchy builder.

Implementation details indicate:

- child tools are rebuilt with restrictions
- recursive spawn/subagent paths are removed from the subagent tool registry

So the intended pattern is:

```text
master -> spawned executor
```

If you want richer coordination, GoClaw expects you to use:

- `team_tasks`
- `team_message`
- lead/member team workflows

So GoClaw prefers **team orchestration** over recursive self-clone trees.

## 3. DeepAgents

In normal DeepAgents usage, the pattern is also mostly:

```text
master -> child
```

The standard `task` tool creates an ephemeral child that:

- runs in isolated context
- returns one final result
- does not behave like a long-lived nested orchestrator by default

You could build more complex nesting by embedding custom graphs or custom agents, but that is **not** the main packaged pattern.

So the practical answer is:

- standard DeepAgents: mostly one child level
- custom DeepAgents + LangGraph: deeper hierarchies are possible

## 4. LangGraph

LangGraph is the most flexible.

It does not impose a fixed nesting depth. You can design:

- one supervisor + one worker
- parallel worker sets
- nested subgraphs
- multi-layer orchestrator-worker trees

So LangGraph is limited mostly by **your graph design**, not by a fixed built-in subagent depth rule.

## Comparison

| System | Default Pattern | Can Child Spawn Child? | Practical Limit |
|--------|------------------|------------------------|-----------------|
| **OpenClaw** | `main -> sub-agent` | Yes, if enabled | Up to sub-sub-agent when `maxSpawnDepth = 2` |
| **GoClaw** | `master -> self-clone worker` | Not the intended `spawn` pattern | Prefer teams instead of recursive spawn |
| **DeepAgents** | `master -> ephemeral child` | Not usually in packaged flow | Possible only through custom graph design |
| **LangGraph** | Whatever you design | Yes | Determined by graph architecture |

## What This Means

There are really two families here:

### Recursive spawn family

- **OpenClaw** supports a bounded recursive spawn model
- **LangGraph** can model arbitrarily nested orchestration

### Supervisor + worker family

- **GoClaw** prefers a lead/team/task-board model over recursive spawn trees
- **DeepAgents** prefers isolated child workers returning results to the parent

## Bottom Line

If your question is:

> "Can a sub-agent become another orchestrator?"

Then:

- **OpenClaw**: yes, but only up to a bounded depth
- **GoClaw**: not really through `spawn`; use team workflows
- **DeepAgents**: not usually in the packaged pattern
- **LangGraph**: yes, if you design it that way

## Sources

- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [LangGraph README](https://github.com/langchain-ai/langgraph/blob/main/README.md)
