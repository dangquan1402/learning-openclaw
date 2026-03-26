# System Message Comparison

> **Scope:** OpenClaw, GoClaw, DeepAgents, and Claude Code

These four systems all use a "system message", but they build it very differently.

## Short version

- **OpenClaw**: compact, section-built runtime prompt with prompt modes and injected workspace files
- **GoClaw**: richest open prompt builder, with many generated sections and role/context files
- **DeepAgents**: smaller base prompt, with behavior added mostly through middleware and tool architecture
- **Claude Code**: the most expanded runtime payload, combining policy, tools, memory, environment, and MCP instructions

## OpenClaw

OpenClaw builds a custom system prompt for every run. Its docs describe a fixed section structure:

- tooling
- safety
- skills
- workspace
- docs
- sandbox
- runtime
- reasoning
- project context injection

It also has prompt modes:

- `full`
- `minimal`
- `none`

Important design choices:

- `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, and `MEMORY.md` can be injected into **Project Context**
- subagents get a smaller prompt and fewer injected files
- the prompt stays relatively compact compared with Claude Code

So OpenClaw treats the system prompt as a **runtime-assembled operating manual plus bootstrap context**.

## GoClaw

GoClaw is similar in style to OpenClaw, but heavier and more explicit.

Its `BuildSystemPrompt()` takes many runtime inputs:

- tool names
- workspace
- channel type
- peer kind
- memory availability
- spawn availability
- team state
- skills summary
- MCP tool descriptions
- sandbox info
- context files
- self-evolution settings
- extra prompt

It also builds many sections:

- persona
- tooling
- safety
- self-evolution
- skills
- MCP
- workspace
- team workspace
- team members
- sandbox
- user identity
- extra context
- project context
- spawn guidance
- runtime
- recency reminders

So GoClaw's system prompt is the most fully developed **open-source control-plane prompt builder** in this comparison.

## DeepAgents

DeepAgents uses a different style.

It has a smaller `BASE_AGENT_PROMPT` that focuses on:

- task completion behavior
- concise communication
- progress updates
- tool use expectations

Then the rest of the behavior is layered through middleware and agent construction:

- todos
- filesystem
- summarization
- memory middleware
- skills middleware
- subagent middleware
- human-in-the-loop middleware

So DeepAgents relies less on one giant prompt and more on **prompt + middleware + graph/tool architecture**.

This is cleaner for framework builders, but less like a giant all-in-one operating document.

## Claude Code

Claude Code is the most expanded and runtime-specific of the four.

From the captured payload you provided, the system content includes:

- base identity
- safety policy
- task-execution guidance
- risky-action guidance
- tool-usage rules
- subagent usage rules
- tone/style rules
- auto memory instructions
- injected `MEMORY.md`
- environment snapshot
- model/version details
- MCP server instructions
- git status snapshot

This is much closer to a **full runtime control document** than a small system prompt.

Important design choices:

- the prompt includes explicit permission/approval behavior
- it injects persistent memory directly
- it includes environment and git state in the system layer
- it includes MCP server instructions inline

So Claude Code's system message is not just a persona or tool guide. It is a **complete execution contract for the session**.

## Main differences

### 1. Prompt size philosophy

- **DeepAgents**: smaller base prompt, more architecture outside the prompt
- **OpenClaw / GoClaw**: section-built runtime prompt
- **Claude Code**: very large runtime system document

### 2. Context injection philosophy

- **OpenClaw / GoClaw**: inject workspace/bootstrap files
- **Claude Code**: inject memory, environment, and runtime state directly
- **DeepAgents**: load more through middleware/backends than a giant prompt blob

### 3. Role/control philosophy

- **OpenClaw**: assistant gateway prompt
- **GoClaw**: workflow/control-plane prompt
- **DeepAgents**: framework behavior prompt
- **Claude Code**: session operating manual

## My judgment

The best pieces from each are:

- **OpenClaw**: compact prompt modes and selective context injection
- **GoClaw**: rich structured section builder
- **DeepAgents**: avoid putting everything into one prompt
- **Claude Code**: explicit operational guidance, permission rules, and runtime context

## What matters most

The important question is not "which system prompt is biggest?"

It is:

- what belongs in the system prompt
- what should live in tools/middleware/state instead
- how much runtime context should be injected directly

## Best synthesis for OpenClaw

If OpenClaw evolves here, the strongest direction is:

- keep prompt modes
- keep compact section-based structure
- avoid Claude-style overexpansion by default
- move some behavior into clearer structured runtime layers
- inject only the most important stable context directly

## Sources

- [OpenClaw System Prompt docs](https://github.com/openclaw/openclaw/blob/main/docs/concepts/system-prompt.md)
- [OpenClaw `system-prompt.ts`](https://github.com/openclaw/openclaw/blob/main/src/agents/system-prompt.ts)
- [GoClaw `systemprompt.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/agent/systemprompt.go)
- [DeepAgents `graph.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/graph.py)
- [Claude Code payload sample](/Users/quandang/Desktop/dquan/products/FuseAPI/full_claude_code_message.json)
