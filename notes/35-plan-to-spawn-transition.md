# Plan-to-Spawn Transition

> **Question:** After an agent makes a plan, how does it actually spawn a subagent? Does it need tool calling, or is there another path?

In practice, planning alone does nothing. The runtime still needs an **execution primitive** to turn "delegate this work" into a real child run.

The common pattern is:

1. the model plans or decides delegation is useful
2. the model emits a **tool-like action**
3. the runtime interprets that action and starts a child run
4. the child eventually returns through a result, announce, or task status path

So yes: in most of these systems, **spawn is effectively implemented as a tool call or tool-equivalent runtime action**.

## OpenClaw

OpenClaw is explicit here.

The agent uses `sessions_spawn` to create a subagent run. That is a normal tool in the session/tool system. The spawn itself is not inferred from plain natural-language output; it happens because the model selects the tool with structured arguments like `task`, `agentId`, `model`, and `thinking`.

So the transition is:

- plan inside the main loop
- emit `sessions_spawn(...)`
- runtime starts the child session
- child announces result back later

## GoClaw

GoClaw follows the same broad pattern.

For self-clone execution, the model uses `spawn` / `sessions_spawn`-style delegation tools. For team workflows, the lead first uses `team_tasks` to create a tracked task, then delegates work to a member through the delegation layer.

So the team path is usually:

- plan
- create task
- delegate with task linkage
- runtime starts the worker/member execution

Again, the runtime does not parse free-form prose like "I will delegate this now" and spawn magically. It needs the structured tool/delegation action.

## DeepAgents

DeepAgents is even more explicit.

Subagents are launched through the `task` tool. Its middleware tells the parent to launch multiple agents concurrently by using multiple tool uses in one message.

So the transition is:

- supervisor plans
- model emits one or more `task(...)` calls
- runtime runs those subagents
- results come back as tool results

## Claude Code

Claude Code is slightly less transparent in the public docs, but the behavior is conceptually the same.

Its docs say Claude can automatically delegate to a subagent when the task matches the subagent's description, or the user can request a specific subagent explicitly.

That "automatic delegation" is still a **runtime action chosen by Claude Code's agent loop**, not just text parsing. In other words:

- Claude plans in the main session
- the agent loop decides to invoke a subagent definition
- the runtime starts that subagent with its own prompt/tools/context

Public docs do not frame this as a user-visible `spawn()` tool the way OpenClaw or DeepAgents do, but architecturally it behaves like an internal delegation tool call.

## What this means

There are two bad mental models to avoid:

- **Wrong:** "the model prints a plan, then the runtime reads the text and guesses what to spawn"
- **Wrong:** "planning itself creates subagents"

The better model is:

- planning chooses an execution mode
- a **structured delegation primitive** performs the spawn

That primitive might be:

- a visible tool (`sessions_spawn`, `task`)
- a structured team delegation action
- an internal subagent runtime call (Claude Code style)

## Bottom line

Yes, some kind of **tool-calling or tool-equivalent structured action** is usually required.

Not because the plan needs to be "parsed" after the fact, but because the runtime needs a concrete, machine-readable instruction to launch a child agent safely and with the right config.

## Sources

- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [GoClaw Agent Teams](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
- [GoClaw Tools System](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/03-tools-system.md)
- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [Claude Code Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- [Claude Code Common Workflows](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/common-workflows)
