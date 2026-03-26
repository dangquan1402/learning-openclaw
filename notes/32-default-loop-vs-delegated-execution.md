# Default Loop vs Delegated Execution

> **Question:** In OpenClaw, GoClaw, and Claude Code, when does the system stay in the main agent loop, and when does it switch to subagents or team execution?

The important pattern is:

1. plan or interpret the task
2. choose an execution mode
3. only delegate when delegation is actually useful

So planning does **not** automatically mean subagents will spawn.

## OpenClaw

OpenClaw stays in the **main agent loop by default**.

It escalates to subagents when work is better treated as:

- research
- long-running work
- slow tool branches
- isolated parallel work

The `sessions_spawn` tool starts a background subagent run, and the result is announced back later. Nested orchestration is optional and only enabled when `maxSpawnDepth >= 2`.

So OpenClaw's model is:

- default main loop first
- subagents only when needed

## GoClaw

GoClaw also has a normal/default loop, but it adds a stronger second layer.

### Layer 1: default loop

The agent can:

- answer directly
- call tools directly
- use `spawn` for a self-clone worker

### Layer 2: team workflow

If team mode is active, the lead agent:

- creates tasks on the shared board
- delegates tasks to members
- receives combined results
- synthesizes the final answer

So GoClaw has both:

- lightweight direct execution
- heavier lead/member workflow execution

## Claude Code

Claude Code also uses the **default session first**.

It delegates to subagents when:

- a task matches a subagent's description
- the user explicitly asks for delegation
- isolated context or specialization is helpful

Todo tracking does not itself imply delegation. A plan or todo list can exist entirely inside the main Claude Code session.

So Claude Code's model is:

- default Claude session first
- subagent delegation when useful or explicitly requested

## Practical takeaway

Across these systems, the real control flow is:

- **plan first**
- then choose one of:
  - stay in main loop
  - call tools directly
  - spawn/delegate to subagents
  - enter a team/workflow mode

This means the important design question is not "do we have planning?"

It is:

> after planning, how does the runtime choose the right execution mode?

## Best interpretation

- **OpenClaw**: main loop with optional subagent escalation
- **GoClaw**: main loop plus a stronger workflow/team layer
- **Claude Code**: main loop plus optional specialized subagents

This is why these systems feel different even when all of them support planning and delegation.

## Sources

- [OpenClaw Sub-Agents docs](https://github.com/openclaw/openclaw/blob/main/docs/tools/subagents.md)
- [GoClaw Agent Teams](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
- [Claude Code Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
- [Claude Code Todo Tracking](https://docs.anthropic.com/en/docs/claude-code/sdk/todo-tracking)
