# Function Calling and Parallel Actions

> **Question:** Can one user input trigger many actions, and is that part of why these systems feel faster?

Yes. But there are **two different acceleration paths**:

1. **Parallel tool calls**: one model turn emits multiple tool calls, and the runtime executes them in parallel.
2. **Parallel subagents**: one model turn delegates independent work to multiple workers with isolated context.

They are related, but not the same architecture.

## OpenClaw

OpenClaw supports provider-level parallel tool calling in its model params layer. The embedded runner resolves and forwards `parallel_tool_calls` when configured.

More importantly, OpenClaw's documented concurrency story is still **subagents**. Its subagent docs explicitly frame them as the way to parallelize research, slow tools, and long-running tasks without blocking the main run.

So for OpenClaw:

- **simple independent actions**: parallel tool calls can help
- **large or slow independent tasks**: subagents are the stronger pattern

## GoClaw

GoClaw has the clearest runtime evidence of **true parallel function calling**.

In `internal/agent/loop.go`, when a response contains more than one tool call, GoClaw:

- emits all `tool.call` events first
- launches each tool in a goroutine
- collects results
- sorts them back into deterministic order
- processes results sequentially afterward

That means one model response can trigger many actions and the runtime actually executes them concurrently.

## DeepAgents

DeepAgents explicitly teaches the parent agent to use **multiple tool uses in one message** when tasks are independent. Its `task` middleware says to launch multiple agents concurrently whenever possible.

But its main optimization story is not only parallel tool calls. It is **parallel `task` subagents**:

- isolate context
- keep intermediate tool noise away from the parent
- return one final result per worker

So DeepAgents is optimized more around **parallel delegation** than around a bare multi-tool loop.

## LangGraph

LangGraph is the lower-level foundation. Its `ToolNode` source executes tool calls in parallel, using `executor.map(...)` in the sync path and `asyncio.gather(...)` in the async path. The same source also notes that the `Send` API is used to distribute tool calls in parallel.

So LangGraph gives the primitives for both:

- parallel tool execution
- parallel graph/subgraph branches

But you design the orchestration yourself.

## Practical takeaway

If you say "one input, many actions", that can mean:

- **tool-call batching** for small independent actions
- **subagent fan-out** for bigger independent tasks

The likely best pattern for OpenClaw is:

- use **parallel tool calls** for short, cheap, direct actions
- use **subagents** for heavy, slow, or context-hungry work

That is faster not only because of concurrency, but because it also reduces unnecessary turns and keeps the main context cleaner.

## Sources

- [OpenClaw `extra-params.ts`](https://github.com/openclaw/openclaw/blob/main/src/agents/pi-embedded-runner/extra-params.ts)
- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [GoClaw `internal/agent/loop.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/agent/loop.go)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [LangGraph `tool_node.py`](https://github.com/langchain-ai/langgraph/blob/main/libs/prebuilt/langgraph/prebuilt/tool_node.py)
