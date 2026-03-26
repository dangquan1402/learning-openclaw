# ACFS Core Loop: Planning, Beads, and Agent Mail

> **Parent:** [Agentic Coding Flywheel Setup (ACFS)](27-agentic-coding-flywheel-setup.md)
> **Tracking:** [Issue #4](https://github.com/dangquan1402/learning-openclaw/issues/4)

## Focus

This note zooms in on the part of ACFS that matters most for agent-team research:

- planning
- beads
- graph-aware routing
- multi-agent coordination

This is the workflow layer on top of the bootstrap stack.

## The Two-Layer Model

ACFS's docs make a sharp distinction between two layers:

### 1. Planning substrate

Use strong frontier models to create and refine a large markdown plan.

This is where:

- product shape
- workflows
- architecture
- edge cases
- testing expectations

get worked out while the whole system still fits in context.

### 2. Core operating loop

Once the plan is ready, ACFS shifts into a smaller durable execution loop built around:

- **Agent Mail**
- **`br`**
- **`bv`**

This is the part ACFS treats as the heart of day-to-day project execution.

## The Core Loop

The workflow can be summarized like this:

1. create a serious markdown plan
2. convert that plan into **beads**
3. attach dependencies between beads
4. use `bv` to identify the highest-value ready bead
5. use Agent Mail for claims, reservations, updates, and completion
6. let agents execute against those beads
7. repeat until the graph is done

This is important because ACFS does not want agents picking work from vague chat memory.

It wants:

- explicit work units
- explicit dependencies
- explicit coordination

## What a Bead Is

A bead is a self-contained unit of work with enough information for an agent to execute without guessing.

From the workflow docs, a good bead includes:

- task context
- why it matters
- dependencies
- definition of done
- test obligations
- likely touch points
- coordination notes

That makes a bead more structured than a normal todo and closer to a task packet for a worker agent.

## Role of `br`

`br` is the task-structure layer.

Its purpose is to:

- encode the plan into explicit work units
- preserve dependencies
- keep work decomposed into pieces agents can claim

So `br` is where the markdown plan becomes executable task structure.

## Role of `bv`

`bv` is the graph-aware routing layer.

Its purpose is to:

- inspect the bead graph
- identify ready work
- prioritize the next highest-leverage bead

This is a meaningful difference from simpler systems that just take the next item in a list.

ACFS is trying to answer:

```text
what is the best unblocked next task?
```

not just:

```text
what is the next numbered task?
```

## Role of Agent Mail

Agent Mail is the coordination layer.

Its purpose is to let agents:

- announce claims
- reserve files or resources
- post progress updates
- hand off work
- complete a bead with a durable thread history

This matters because ACFS assumes multiple agents may be active at once.

Without explicit communication, they risk:

- file collisions
- duplicated effort
- invisible progress
- weak handoff

## Why `AGENTS.md` Still Matters

ACFS also treats `AGENTS.md` as a key part of the loop.

The reason is practical:

- long-running agent sessions compact
- swarms need shared operating rules
- fresh or resumed agents need to rehydrate the same expectations

So `AGENTS.md` is not just setup documentation. It is an operational memory artifact.

## What Makes This Different

Compared with simpler supervisor/worker systems, ACFS pushes harder on:

- explicit planning before execution
- explicit task packets
- dependency graph routing
- explicit worker coordination rituals

That makes it more swarm-oriented than many "just spawn a subagent" designs.

## Implications for OpenClaw

The strongest reusable ideas here are:

- separate planning from execution
- make task units explicit and inspectable
- add claim/progress/handoff semantics
- consider graph-aware routing when dependencies matter
- preserve an `AGENTS.md`-like operating contract for long-lived workers

OpenClaw does not need ACFS's whole bootstrap stack to benefit from these ideas.

## Bottom Line

The ACFS core loop is:

- **planning-first**
- **task-graph driven**
- **coordination-heavy**

Its key contribution is not one specific binary. It is the combination of:

- plan
- bead graph
- routing
- agent communication

as one coherent swarm workflow.

## Sources

- [The Flywheel Core Loop](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_CORE_LOOP.md)
- [The Flywheel Approach to Planning and Beads Creation](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_APPROACH_TO_PLANNING_AND_BEADS_CREATION.md)
