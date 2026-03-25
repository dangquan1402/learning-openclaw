# DeepAgents: Subagent Architecture and Output Control

> **Parent:** [DeepAgents Investigation](12-deepagents.md)  
> **Repo:** [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents)

## What This Note Focuses On

This note goes deeper on how DeepAgents handles:

- synchronous subagents
- asynchronous subagents
- output isolation
- backend/shared-state design

DeepAgents is not a team product like GoClaw. It is a **framework-level supervisor pattern**.

## 1. Sync Subagents: The `task` Tool

DeepAgents exposes a `task` tool that launches **ephemeral subagents**.

The middleware guidance is very explicit:

- launch multiple subagents concurrently when tasks are independent
- give each child a detailed, autonomous objective
- the child returns **one final message**
- that result is **not visible to the user**
- the parent agent must summarize or integrate it into the final user response

This is a strong design choice. DeepAgents intentionally hides the child trajectory and keeps only the final report.

## 2. Why the Isolation Matters

The `task` tool is designed to prevent the main thread from getting overloaded with:

- long research traces
- tool call noise
- huge intermediate outputs
- irrelevant context from sibling tasks

The middleware excludes several state keys when invoking children and when returning state from them. The main principle is:

> subagents should do heavy work in isolation, then hand back a compact final result.

That makes the parent agent the clear owner of final presentation.

## 3. Parallel Sync Subagents

DeepAgents strongly encourages parallelism for independent tasks.

The tool description says to launch multiple agents concurrently whenever possible. The examples show classic supervisor behavior:

- one child per research target
- one child per meeting agenda
- one child for heavy repo analysis

This is closer to a **token-efficiency and context-isolation** strategy than a durable team workflow.

## 4. Async Subagents

DeepAgents also has **async subagents**, but the model is different from local sync children.

Async subagents:

- run on **remote LangGraph servers**
- return a `task_id` immediately
- persist tracked task state in the parent agent state
- are controlled with explicit tools:
  - `start_async_task`
  - `check_async_task`
  - `update_async_task`
  - `cancel_async_task`
  - `list_async_tasks`

The docs are strict about behavior:

- after launching, return control to the user immediately
- do not auto-poll
- only check status when the user asks
- never trust stale task status from earlier conversation history

This makes async subagents feel more like **remote jobs** than local child agents.

## 5. Backends and Shared State

One of DeepAgents' most important ideas is the backend layer.

Backends let the parent and child agents share or isolate their working context in different ways:

- filesystem backends
- sandbox backends
- state backend using LangGraph state/checkpoints
- composite backends

The `StateBackend` is especially important conceptually:

- files live inside LangGraph state
- they persist within a conversation thread
- they are checkpointed after steps
- they do not become cross-thread long-term memory automatically

This lets subagents create artifacts that the parent can later inspect without stuffing raw content into the parent message history.

## 6. What the Parent Actually Controls

DeepAgents gives the supervisor strong control in three ways:

### Output filtering

The parent gets the child's **final message**, not the full internal tool trace.

### Prompt-level control

The parent decides:

- which subagent type to use
- what instructions the child receives
- what output format the child must return

### State-level control

The parent decides whether collaboration happens through:

- final text only
- shared files/backends
- remote async tasks on LangGraph deployments

This is cleaner than many multi-agent systems because the control boundaries are deliberate.

## 7. What DeepAgents Does Not Try to Be

DeepAgents is not trying to be:

- a task board
- a multi-tenant team dashboard
- a role-based lead/member workflow product
- a messaging-native assistant platform

That is the main contrast with GoClaw. DeepAgents is about **good supervisor mechanics**, not full organization-level team operations.

## 8. Why This Matters

DeepAgents shows a strong opinionated answer to the "master agent controls worker output" problem:

- children work privately
- the parent receives only final reports
- async jobs have explicit lifecycle tools
- shared state is handled through backends instead of ad hoc memory leaks

For coding and research agents, this is a very clean design.

## Bottom Line

DeepAgents treats subagents as **isolated execution units** rather than visible collaborators.

That means the parent agent stays in control of:

- when to delegate
- how many children to launch
- what each child must return
- whether to use shared files/state
- what the user finally sees

It is one of the clearest framework implementations of supervisor-controlled subagent output.

## Sources

- [DeepAgents Subagents docs](https://docs.langchain.com/oss/python/deepagents/subagents)
- [DeepAgents Async Subagents docs](https://docs.langchain.com/oss/python/deepagents/async-subagents)
- [DeepAgents Backends docs](https://docs.langchain.com/oss/python/deepagents/backends)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [DeepAgents `async_subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/async_subagents.py)
- [DeepAgents `state.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/backends/state.py)
