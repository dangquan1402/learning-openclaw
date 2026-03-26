# Claude Code Memory and Tool Model

> **Question:** How does Claude Code handle memory, tools, permissions, and subagent capabilities at a deeper level?

Claude Code uses a layered model. It does not treat "memory" as one thing.

## 1. Project memory vs auto memory

Claude Code starts each session with a fresh context window, then reloads durable context from two systems:

- **`CLAUDE.md` files**: explicit instructions written by users or teams
- **auto memory**: notes Claude writes for itself based on useful learnings across sessions

`CLAUDE.md` files are loaded by walking up the directory tree from the current working directory. They can also import other files with `@path` syntax, and larger projects can organize rules under `.claude/rules/`.

Auto memory is separate. It is enabled by default and stores project-specific notes under:

- `~/.claude/projects/<project>/memory/`

That directory has a `MEMORY.md` index plus optional topic files. This is Claude's own accumulated memory about the project: build commands, debugging notes, architecture facts, code style preferences, and workflow habits.

## 2. Subagent memory

Claude Code also supports **persistent memory per subagent**.

A subagent can declare:

- `memory: user`
- `memory: project`
- `memory: local`

When enabled, the subagent gets:

- a persistent memory directory
- memory instructions injected into its prompt
- the first 200 lines of `MEMORY.md` loaded into context
- `Read`, `Write`, and `Edit` enabled automatically so it can maintain that memory

This is a notable design choice: memory is not only global or project-level. It can belong to a specific agent persona.

## 3. Tool inheritance and restriction

By default, subagents inherit all tools from the main conversation, including MCP tools.

But Claude Code lets you narrow that with:

- `tools`: allowlist
- `disallowedTools`: denylist

It can also restrict which subagents a coordinator can spawn using:

- `Agent(worker, researcher)`

So Claude Code treats tool access as part of the agent definition, not just a global runtime setting.

## 4. MCP, skills, and plugins

Claude Code's tool surface has multiple layers:

- **internal tools**
- **MCP servers** for external systems and APIs
- **skills** as reusable prompt playbooks
- **plugins** that can package skills, agents, hooks, and MCP servers

Skills are especially important because they are not just shortcuts. A skill can load supporting files, inject dynamic context, and even run work in a subagent. Bundled skills like `/batch` can decompose work, present a plan, then spawn background agents in isolated git worktrees.

## 5. Permission architecture

Claude Code separates tool capability from permission approval.

Its permission system is tiered:

- read-only tools: no approval
- bash commands: approval required
- file modification: approval required

Permissions can be managed with `/permissions`, checked into settings, and refined with hooks. `PreToolUse` hooks can approve or deny tool calls dynamically before the standard permission system runs.

## 6. What is special here

Claude Code's strength is not just "it has subagents." The special part is the combination of:

- explicit project memory
- automatic learned memory
- agent-specific persistent memory
- scoped tools and MCP access
- skills as reusable execution playbooks
- permission rules that are separate from tool definitions

That gives it a strong control model:

- default session for ordinary work
- subagents for specialized work
- memories at multiple scopes
- tools and permissions controlled independently

## 7. What OpenClaw could borrow

The most valuable ideas for OpenClaw are:

- **auto memory** separate from static project instructions
- **agent-specific memory** for specialized workers
- **tool scoping per agent** instead of only global tool policy
- **skills as reusable execution playbooks**
- **permissions as a separate policy layer**

This is one of the cleanest examples of a modern coding-agent control plane.

## Sources

- [Claude Code Memory](https://code.claude.com/docs/en/memory)
- [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Settings](https://code.claude.com/docs/en/settings)
- [Claude Code MCP](https://code.claude.com/docs/en/mcp)
- [Claude Code Skills](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Team / Permissions](https://code.claude.com/docs/en/team)
