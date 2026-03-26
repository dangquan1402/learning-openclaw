# Memory Model Comparison

> **Question:** How do OpenClaw, GoClaw, and DeepAgents handle memory?

They solve different memory problems.

- **OpenClaw**: file-based assistant memory
- **GoClaw**: retrieval-backed operational memory
- **DeepAgents**: programmable memory architecture

## OpenClaw

OpenClaw's memory model is the simplest to understand.

It combines:

- workspace files
- session history
- compaction / pruning
- optional memory plugins and search

In practice, the model is centered on Markdown memory in the agent workspace, such as `MEMORY.md` and `memory/YYYY-MM-DD.md`, plus session-level context management to keep the active prompt small.

This is a good fit for:

- local-first assistants
- human-readable memory
- agent workflows where files are the source of truth

Its weakness is that memory is less structured and less retrieval-heavy than GoClaw.

## GoClaw

GoClaw has the strongest built-in long-term memory layer of the three.

It combines:

- per-user and per-agent context files
- `MEMORY.md` and `memory/*.md`
- `memory_search` and `memory_get`
- automatic indexing into Postgres-backed memory tables
- hybrid retrieval (BM25 + vector)
- knowledge graph search

An important detail is that writing a Markdown memory file automatically triggers indexing.

This makes GoClaw's memory feel less like "notes on disk" and more like a **retrieval system with scoped memory documents**.

This is a strong fit for:

- enterprise or multi-user deployments
- durable recall across sessions
- workflows where memory must be searchable and operational

## DeepAgents

DeepAgents has the most flexible design.

It separates:

- **working memory**: thread state, todos, filesystem context
- **long-term memory**: files loaded through `MemoryMiddleware`
- **durable storage**: optional store-backed backends

Its default `StateBackend` keeps files in LangGraph state for a thread. More advanced backends can make some paths persistent, such as durable `/memories/` storage while keeping scratch files ephemeral.

So DeepAgents is less a single memory product and more a **memory architecture kit**.

This is a strong fit for:

- framework builders
- agents that need clear scratchpad vs durable-memory separation
- systems where memory policy should be programmable

## Practical takeaway

These systems represent three different philosophies:

- **OpenClaw**: memory as workspace + session management
- **GoClaw**: memory as indexed retrieval + scoped operational context
- **DeepAgents**: memory as backend-controlled state and file layers

## Best by use case

- **Best simple model**: OpenClaw
- **Best team / enterprise memory model**: GoClaw
- **Best framework memory model**: DeepAgents

If OpenClaw wants to improve here, the biggest opportunity is to keep its simple file-first model but add:

- stronger indexed recall
- clearer separation between scratch memory and durable memory
- optional structured memory beyond plain Markdown

## Sources

- [OpenClaw README](https://github.com/openclaw/openclaw/blob/main/README.md)
- [OpenClaw Docs Index](https://github.com/openclaw/openclaw/blob/main/docs/index.md)
- [GoClaw Tools System](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/03-tools-system.md)
- [GoClaw Bootstrap, Skills, Memory](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/07-bootstrap-skills-memory.md)
- [GoClaw Agent Loop](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/01-agent-loop.md)
- [DeepAgents README](https://github.com/langchain-ai/deepagents/blob/main/README.md)
- [DeepAgents `graph.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/graph.py)
