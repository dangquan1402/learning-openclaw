# ACFS Task and Agent Management

> **Parent:** [Agentic Coding Flywheel Setup (ACFS)](27-agentic-coding-flywheel-setup.md)
> **Tracking:** [Issue #4](https://github.com/dangquan1402/learning-openclaw/issues/4)

## Focus

This note ignores most setup details and focuses only on how ACFS manages:

- tasks
- agent work selection
- coordination
- concurrency
- human oversight

## Core Model

ACFS treats task and agent management as a combination of:

- a centralized **bead graph**
- decentralized **agent choice and execution**
- explicit **message-based coordination**

The clearest summary from the workflow material is:

- the **beads themselves are centralized**
- the **agents are distributed**
- agents use **`bv`** to decide what to work on
- agents coordinate through **Agent Mail**

So ACFS is not mainly a master-agent assigning every step. It is closer to a swarm working from shared task structure.

## Task Unit: Beads

The core task abstraction is the **bead**.

Beads are meant to be:

- self-contained
- dependency-aware
- execution-ready

The docs and workflow posts suggest that once the plan is converted into beads properly, agents should not need the full master plan all the time.

That is important:

- the plan is the design artifact
- the beads become the execution artifact

## Task Ordering

ACFS does not primarily use simple list ordering.

Instead:

- dependencies are encoded into the bead graph
- `bv` is used as a kind of routing compass
- agents pick the best ready bead rather than the next numbered line item

This makes ACFS much more graph-driven than queue-driven.

## Agent Coordination

Agent Mail is the coordination layer.

Agents use it for:

- claims
- reservations
- progress messages
- handoff
- completion

This matters because ACFS assumes many agents may be running in the same shared workspace.

The coordination system is there to reduce:

- duplicated effort
- task collisions
- file collisions
- invisible work

## Shared Workspace Assumption

One notable ACFS preference is that agents often work in the **same shared workspace** rather than isolated worktrees.

The source material explicitly pushes back on worktrees in early swarm stages because the system wants agents sharing the same current state.

That means ACFS relies more heavily on:

- reservations
- coordination threads
- bead state

than on branch/worktree isolation.

## Human Role

The human is not meant to micromanage every implementation step.

The human's main leverage is earlier:

- creating the initial markdown plan
- improving it aggressively
- cross-checking beads against the plan
- making sure the bead graph is complete before the swarm starts

During execution, the human tends the system more than they hand-assign every step.

## Review Loops

ACFS also uses agents for review, not just implementation.

The workflow posts describe:

- self-review
- cross-review of other agents' work
- repeated review rounds
- test creation

So the task system is not just "claim bead -> code -> done".

It is closer to:

- claim
- implement
- review
- test
- revisit if needed

## Concurrency Behavior

ACFS documents some practical concurrency issues directly:

- agents should mark beads in progress quickly
- start times should be staggered
- this helps avoid a "thundering herd" where many agents grab the same work

This is a useful sign that the system is designed around real swarm behavior, not just idealized delegation.

## What This Means Architecturally

ACFS task management is best described as:

- **centralized task graph**
- **decentralized worker choice**
- **explicit coordination threads**
- **shared-workspace swarm execution**

It is not a classic manager-worker scheduler where one supervisor assigns every ticket one by one.

## Relevance to OpenClaw

If OpenClaw wants richer task/agent management, ACFS suggests a path like:

- keep a durable task graph
- let workers self-select from ready tasks
- add explicit claim/progress/handoff communication
- handle collision risk with reservations and rapid state updates
- preserve a shared execution contract through artifacts like `AGENTS.md`

## Bottom Line

ACFS manages tasks and agents through a **swarm coordination model**, not a simple queue.

The key idea is:

- formalize tasks centrally
- let agents navigate that structure semi-autonomously
- use explicit messaging to keep the swarm coherent

## Sources

- [The Flywheel Core Loop](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_CORE_LOOP.md)
- [The Flywheel Approach to Planning and Beads Creation](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_APPROACH_TO_PLANNING_AND_BEADS_CREATION.md)
- [Complete X Posts About Planning and Beads](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/COMPLETE_X_POSTS_ABOUT_PLANNING_AND_BEADS.md)
