# Execution-Mode Selection

> **Question:** Across OpenClaw, GoClaw, DeepAgents, and Claude Code, how does the system choose between staying in the main loop, calling tools directly, spawning subagents, or using a workflow/team mode?

This is one of the most important missing-piece layers.

Planning is not enough. After planning, the system still needs to choose the **execution mode**.

## The four execution modes

Most of these systems end up selecting between four patterns:

1. **Main loop**: answer directly in the current agent session
2. **Direct tools**: call tools in the current session
3. **Subagents**: delegate isolated work to child workers
4. **Workflow/team mode**: create durable tasks and coordinate multiple workers over time

The strongest systems do not force everything through one mode.

## OpenClaw

OpenClaw is closest to:

- main loop first
- direct tools for normal work
- `sessions_spawn` for heavy or parallel branches

It already has a good escalation path, but the policy is still relatively light. The decision logic is mostly expressed through prompt/runtime conventions, not a richer visible planning/control layer.

### What is missing

- clearer criteria for when to stay in main loop vs spawn
- stronger progress visibility after escalation
- better distinction between "parallel tool calls" and "parallel subagents"

## GoClaw

GoClaw has the broadest execution spectrum:

- main loop
- direct tool calling
- self-clone `spawn`
- full team workflow with shared board

This is powerful because the system can move from lightweight work to heavy workflow mode. But it is also easier to overuse the heavy layer.

### What is missing

- a cleaner default policy so ordinary tasks stay lightweight
- stronger guardrails against unnecessary workflow escalation

## DeepAgents

DeepAgents is strong at the middle of the spectrum:

- direct tools for simple actions
- `task` subagents for complex isolated branches

It is one of the clearest implementations of supervisor-style execution-mode selection.

### What is missing

- a richer workflow/team layer for durable coordination
- more visible product-level execution state for non-framework users

## Claude Code

Claude Code has the cleanest overall control model for coding-agent work:

- default main session first
- direct tools when simple
- subagents when specialization or context isolation is useful

It appears to have the best practical instinct for not over-delegating.

### What is missing

- less visible workflow mode than GoClaw
- less transparent delegation policy than fully open frameworks

## What matters most

The key quality is not "how many modes exist." It is **how well the system chooses between them**.

The best selection policy usually looks like this:

- **main loop** if the answer is simple
- **direct tools** if actions are short and local
- **parallel tool calls** if multiple simple actions are independent
- **subagents** if the branch is long, noisy, or context-heavy
- **workflow mode** only if the work is durable, reviewable, or dependency-heavy

## The main missing piece

Most systems still under-specify the decision policy.

They often have the mechanisms, but not a clearly surfaced rule set for:

- when to escalate
- how far to escalate
- when to stay simple

That missing policy layer affects:

- latency
- cost
- user trust
- orchestration quality

## What OpenClaw should improve

For OpenClaw, the biggest improvement is not "more execution modes." It already has enough primitives.

The higher-value improvement is:

- explicit execution-mode selection policy
- visible progress after escalation
- cleaner separation of:
  - direct tools
  - parallel tools
  - subagents
  - optional workflow mode

## Bottom line

Execution-mode selection is a first-class architecture problem.

The strongest design is:

- lightweight by default
- escalated only when justified
- durable only when necessary

That is the right standard for evaluating the other missing layers too.
