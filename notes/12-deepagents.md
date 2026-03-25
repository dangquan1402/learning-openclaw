# DeepAgents: Investigation

> **Repo:** [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents) | **Docs:** [LangChain Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview) | **License:** MIT

## What Is It?

An open-source **agent harness** from LangChain for building deeper, longer-horizon agents without wiring everything manually. It is not a chat app like Open WebUI and not a messaging gateway like OpenClaw. It is a **Python SDK + CLI** that bundles the patterns used by tools like Claude Code and Deep Research into reusable defaults.

| Metric | Value |
|--------|-------|
| Stars | ~17.4K |
| Forks | ~2.4K |
| Default branch | `main` |
| Latest release | `deepagents==0.5.0a2` (Mar 23, 2026) |
| Language | Python |
| Built on | LangChain + LangGraph |
| Positioning | "Batteries-included agent harness" |

## Core Idea

DeepAgents argues that simple tool-calling loops are too shallow for complex work. Its default design combines four patterns:

1. **Planning** via a built-in `write_todos` tool
2. **Filesystem context** via tools like `read_file`, `write_file`, `edit_file`, `ls`, `glob`, `grep`
3. **Subagents** via a `task` tool that delegates work with isolated context windows
4. **Detailed system prompts** that teach the model how to use these tools effectively

This gives you a working deep agent immediately, then lets you customize models, tools, prompts, memory, and sandbox backends later.

## Repository Structure

```text
README.md             Overview and quickstart
examples/             Deep research, content builder, text-to-SQL, autonomous loops
libs/deepagents/      Core SDK
libs/cli/             Terminal coding agent / TUI
libs/evals/           Evaluation suite and benchmark runners
libs/partners/        Sandbox provider integrations
libs/acp/             Agent Client Protocol integration
```

This is a **real product repo**, not just a demo. The presence of a CLI, eval harness, examples, and partner sandboxes makes it closer to a framework ecosystem.

## Key Capabilities

### SDK

- `create_deep_agent()` returns a compiled LangGraph agent
- Works with any tool-calling model
- Supports custom tools, custom prompts, MCP adapters, and long-running tasks

### CLI

- Terminal coding agent similar to Claude Code or Cursor
- Interactive TUI with streaming output
- Web search support
- Headless mode for scripting/CI
- Human-in-the-loop approval for tool calls
- Persistent memory and custom skills

### Evals

The eval package is unusually mature:

- end-to-end trajectory scoring
- LangSmith tracing
- tests for file tools, memory, summarization, skills, and subagent behavior
- benchmark runners for Harbor and Terminal Bench 2.0

This is one of the strongest signals that the project is being built for serious agent engineering, not only demos.

## Architecture Pattern

```text
User task
   ↓
DeepAgents system prompt + planning middleware
   ↓
LangGraph runtime
   ↓
Tools: filesystem, shell, web, custom tools, subagents
   ↓
Optional memory, sandbox, and eval infrastructure
```

The important idea is that the **filesystem becomes working memory**. Instead of forcing everything into one context window, the agent can save intermediate work to files, reload only what it needs, and hand off scoped tasks to subagents.

## How It Compares to OpenClaw

| Area | DeepAgents | OpenClaw |
|------|------------|----------|
| **Primary shape** | Agent harness / SDK / CLI | Always-on messaging agent platform |
| **Core runtime** | LangGraph graph | OpenClaw Gateway + Brain |
| **Main UX** | Terminal coding and research agent | Cross-channel assistant in WhatsApp, Telegram, Slack, etc. |
| **Planning** | Built-in todo tool | Reasoning loop + heartbeat/task patterns |
| **Subagents** | Explicit built-in delegation tool | Multi-agent patterns exist, but different product focus |
| **Memory** | Filesystem + persistent memory options | Markdown memory system and local context |
| **Deployment model** | Library, CLI, sandbox integrations | Self-hosted gateway with Control UI and channels |

## Why It Matters for This Repo

DeepAgents is useful as a **comparison target for agent architecture**, especially:

- planning tools versus pure ReAct loops
- filesystem-as-context patterns
- context isolation through subagents
- eval-driven agent development
- terminal coding agent design

It is **not** a direct competitor to OpenClaw's messaging gateway model. The better framing is: OpenClaw is an always-on assistant product; DeepAgents is an agent-building toolkit.

## Honest Assessment

### Strengths

- Strong default architecture for coding/research agents
- Clear productization beyond the core SDK
- Serious eval investment
- Provider-agnostic and open source

### Limits

- Less relevant if your main interest is multi-channel messaging automation
- More framework-oriented than ready-to-use assistant platform
- Security model is "trust the LLM"; boundaries must be enforced at tool/sandbox level

## Recommendation

Worth keeping in this knowledge base as an **ecosystem comparison note**. The most valuable angle is not feature copying, but understanding why LangChain standardized on planning + files + subagents + evals as the default recipe for deeper agents.

## Sources

- [GitHub repository](https://github.com/langchain-ai/deepagents)
- [Main README](https://github.com/langchain-ai/deepagents/blob/main/README.md)
- [Deep Agents overview docs](https://docs.langchain.com/oss/python/deepagents/overview)
- [CLI README](https://github.com/langchain-ai/deepagents/blob/main/libs/cli/README.md)
- [Evals README](https://github.com/langchain-ai/deepagents/blob/main/libs/evals/README.md)
- [Release list](https://github.com/langchain-ai/deepagents/releases)
