# Should OpenClaw Adopt Todo Tracking, a Masterboard, or Both?

> **Focus:** Design recommendation for OpenClaw based on Claude Code, DeepAgents, GoClaw, and LangGraph patterns.

## Short Answer

**OpenClaw should probably adopt both, but as two separate layers.**

Not every workflow needs a masterboard. But almost every non-trivial agent workflow benefits from todo tracking.

So the recommendation is:

- **Todo tracking** as the default lightweight planning/progress layer
- **Masterboard** only for real multi-agent collaboration

## Why These Should Be Separate

The research shows two distinct needs:

### 1. Personal execution tracking

This is the "what am I doing right now?" layer.

Good fit for:

- a single agent
- a master agent planning its own work
- progress visibility for long tasks
- compact execution state

This is where **Claude-style todo tracking** fits.

### 2. Shared collaboration tracking

This is the "who owns what, what is blocked, what is finished, what depends on what?" layer.

Good fit for:

- lead + worker teams
- direct teammate communication
- task claiming / reassignment
- dependencies / blockers / review
- durable multi-agent coordination

This is where **GoClaw-style task board / masterboard** fits.

## Recommendation for OpenClaw

## Phase 1: Add Todo Tracking First

This is the highest-value, lowest-complexity upgrade.

OpenClaw should add:

- `pending / in_progress / completed` todo lifecycle
- streamed visibility to the user or Control UI
- per-run or per-session todo state
- lightweight agent-facing tool or runtime primitive for updating todos

Why first:

- useful even without subagents
- helps planning and user trust
- much cheaper than building team coordination
- aligns with Claude Code and DeepAgents-style supervisor patterns

## Phase 2: Add Masterboard Only for Team Workflows

OpenClaw should add a shared board only if it wants a serious **lead/team** story.

That board should support:

- shared task list
- task assignment / claiming
- dependencies
- blocker/escalation state
- lead-side synthesis after worker completion
- durable storage outside one prompt window

Why second:

- a board adds real complexity
- it is unnecessary for simple parent-child delegation
- the value appears only when workers must coordinate with each other or with humans

## What Other Systems Suggest

### Claude Code

- uses **todo tracking** for lightweight structured progress
- uses **agent teams** with a shared task list only for heavier collaborative workflows

This strongly supports the "both, but separate" model.

### DeepAgents

- strongly benefits from todo/planning
- does not really require a heavy masterboard for its main supervisor/subagent pattern

This suggests todo tracking has wider applicability than a board.

### GoClaw

- shows the value of a board once you want true team workflows
- but also shows how much more machinery is required: task state, mailbox, escalation, UI, events

This suggests a board should be treated as a **major feature**, not a small extension.

## Proposed Architecture

```text
Layer 1: Todo tracking
  per-agent
  lightweight
  execution/progress visibility

Layer 2: Masterboard
  shared across agents
  durable coordination state
  used only for collaborative team workflows
```

## My Recommendation

If OpenClaw asks "what should we build next?", I would say:

1. **Build todo tracking first**
2. **Keep current subagent announce model**
3. **Only build a masterboard if OpenClaw wants first-class agent teams**

That gives the best cost/benefit sequence.

## Bottom Line

**Yes, do both. But do not treat them as the same feature.**

- Todos solve planning and progress visibility
- Masterboard solves shared multi-agent coordination

The right architecture is not "board instead of todos".

It is:

> todos for every serious agent, board only for true teams.

## Related Notes

- [Claude Code Todo Tracking](22-claude-code-todo-tracking.md)
- [Claude Code Agent Teams](21-claude-code-agent-teams.md)
- [GoClaw Team Execution](15-goclaw-team-execution.md)
- [DeepAgents Subagent Architecture](16-deepagents-subagent-architecture.md)
- [Master Agent Control Comparison](14-master-agent-team-control.md)
