# Claude Code Agent Teams: Investigation

> **Repo/Product:** Claude Code  
> **Docs:** [Agent teams](https://code.claude.com/docs/en/agent-teams) | [Subagents](https://code.claude.com/docs/en/sub-agents) | [Settings](https://code.claude.com/docs/en/settings)

## What It Is

Claude Code has **two different multi-agent modes**:

1. **Subagents**
2. **Agent teams**

This distinction matters.

### Subagents

Subagents are specialized workers inside a single main session:

- separate context window
- custom system prompt
- tool restrictions
- independent permissions
- results returned to the main Claude session

Anthropic is explicit that subagents are for focused work where only the result matters.

### Agent teams

Agent teams are a heavier coordination mode:

- one **lead** session
- multiple **teammate** sessions
- a **shared task list**
- a **mailbox / direct teammate messaging** system
- teammates can communicate directly with each other

This is much closer to a real team workflow than normal subagents.

## Important Differences from Claude Code Subagents

Anthropic draws the line clearly:

- **Subagents**: work inside a single session and report back to the main agent
- **Agent teams**: coordinate across **separate sessions**

Also:

- subagents **cannot spawn other subagents**
- if you need parallel workers that communicate with each other, Anthropic says to use **agent teams**

So Claude Code itself already separates:

```text
subagents = focused workers
agent teams = collaborative multi-session workflow
```

## Architecture

Anthropic describes an agent team as:

- **Team lead**: the main Claude Code session that spawns teammates and coordinates work
- **Teammates**: separate Claude Code instances with their own context windows
- **Task list**: shared list of tasks with task states
- **Mailbox**: messaging system between agents

Stored locally:

- team config: `~/.claude/teams/{team-name}/config.json`
- task list: `~/.claude/tasks/{team-name}/`

So yes, Claude Code effectively has something very close to a **masterboard**:

- not called that by Anthropic
- but functionally it is a **shared task board / task list**

## Task Coordination

Tasks have explicit states:

- `pending`
- `in_progress`
- `completed`

They can also have dependencies, and Anthropic states:

- blocked tasks unblock automatically when dependencies complete
- teammates can self-claim tasks
- the lead can assign tasks directly
- task claiming uses **file locking** to avoid race conditions

This is stronger than plain subagent delegation and closer to a real workflow engine.

## Communication Model

Claude Code agent teams support:

- automatic message delivery between teammates
- idle notifications when a teammate finishes
- direct messaging to a specific teammate
- broadcast to all teammates

This is a real collaborative model, not just parent-child delegation.

## Context and Permissions

Each teammate:

- has its own context window
- loads normal project context such as `CLAUDE.md`, MCP servers, and skills
- does **not** inherit the lead's conversation history

Permissions:

- teammates start with the lead's permission settings
- after spawn, per-teammate mode can change

So Claude Code teams are not just summary-return workers. They are closer to independent agents sharing a coordination layer.

## Compare with DeepAgents

DeepAgents is closer to **Claude Code subagents** than to Claude Code agent teams.

Why:

- isolated worker contexts
- parent-controlled delegation
- results summarized back to parent
- no first-class shared team board in the same sense

DeepAgents async tasks add job tracking, but not the same collaborative team layer.

## Compare with GoClaw

GoClaw teams are the closest architectural match to Claude Code agent teams.

Shared ideas:

- one lead / multiple workers
- shared task list / task board
- teammate communication
- explicit coordination layer
- lead synthesis of results

The main difference is implementation style:

- **Claude Code** stores team/task artifacts locally and currently marks teams as experimental
- **GoClaw** pushes harder into a more formal backend-centered workflow model with PostgreSQL, events, and dashboard surfaces

## Do We Need a Masterboard?

Claude Code suggests a useful answer:

- for normal subagent delegation: **no**
- for true collaborative teams: **yes, or something close to it**

Claude Code itself evolved from simple subagents to a model that includes:

- shared task list
- teammate messaging
- coordination across sessions

That strongly suggests a board-like structure becomes valuable once agents need to:

- self-coordinate
- claim work
- wait on dependencies
- communicate directly
- persist team state outside one master prompt

## Bottom Line

Claude Code has both architectures:

- **subagents** for lightweight supervisor/worker delegation
- **agent teams** for collaborative multi-session work with a shared task list

So if the question is whether a masterboard is necessary:

- not for simple subagents
- but once you want real agent teamwork, Claude Code's design suggests that **some kind of shared task board is very useful**

## Sources

- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Settings](https://code.claude.com/docs/en/settings)
