# Claude Code Todo Tracking: Investigation

> **Docs:** [Todo Lists](https://platform.claude.com/docs/en/agent-sdk/todo-tracking) | [Agent loop](https://platform.claude.com/docs/en/agent-sdk/agent-loop)

## What It Is

Claude's Agent SDK includes built-in **todo tracking** as part of the normal agent loop.

This is lighter than a full team task board. It is mainly a way to:

- organize multi-step work
- show progress to users
- keep the agent's current plan visible

It is not the same thing as Claude Code **agent teams**, which use a shared task list across multiple teammate sessions.

## Todo Lifecycle

Anthropic documents a simple lifecycle:

1. `pending`
2. `in_progress`
3. `completed`
4. removed when the whole group is done

This is much closer to a **progress checklist** than a workflow engine.

## When Claude Uses Todos

The SDK automatically creates todos for:

- complex multi-step tasks
- user-provided task lists
- non-trivial operations that benefit from progress tracking
- explicit requests for todo organization

So todo tracking is not always-on project management. It is a lightweight execution aid inside the agent loop.

## How It Appears in the Runtime

Anthropic exposes todo changes through the message stream using the `TodoWrite` tool.

That means:

- the agent updates todos during the loop
- clients can observe those updates in real time
- a UI can render progress as the task evolves

This is important: todo tracking is part of **runtime execution visibility**, not just a static markdown checklist.

## Compare with Agent Teams

Claude Code has two levels of task coordination:

### A. Todo tracking

- per-agent progress tracking
- lightweight
- no teammate claiming
- no shared collaboration layer
- good for "what am I doing now?"

### B. Agent team task list

- shared across teammates
- task claiming
- dependencies
- teammate coordination
- closer to a real task board

So the todo list is not the same thing as the agent-team task list.

## Compare with GoClaw

GoClaw's team task board is much heavier:

- multi-agent shared board
- ownership/claiming
- blocker handling
- dependencies
- lifecycle events
- durable backend state

Claude todo tracking is much lighter:

- personal execution tracking
- progress visibility
- not a full team workflow layer

So Claude todo tracking is closer to:

- a **working plan**
- or a **live checklist**

than to a full **masterboard**.

## Compare with DeepAgents

DeepAgents has planning/todo concepts, but Claude's SDK todo tracking is notable because it is surfaced explicitly as `TodoWrite` events in the stream.

That makes it useful for:

- live progress UIs
- structured step visibility
- showing users what the agent is currently doing

This is a good fit for single-agent or supervisor-style workflows, even if you never build full multi-agent teams.

## Do We Need This?

My view:

- **Todo tracking** is useful almost everywhere
- **shared task boards** are only needed for true collaborative teams

That means a good architecture can have both:

- todo list for the current agent's execution
- shared board for multi-agent collaboration

Claude's docs strongly suggest these should be treated as **different layers**, not as the same feature.

## Bottom Line

Claude todo tracking is:

- lightweight
- execution-focused
- stream-visible
- useful for progress reporting

It is **not** a replacement for a shared team board, but it is a very useful lower-level building block.

## Sources

- [Claude Agent SDK Todo Lists](https://platform.claude.com/docs/en/agent-sdk/todo-tracking)
- [Claude Agent SDK Agent Loop](https://platform.claude.com/docs/en/agent-sdk/agent-loop)
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
