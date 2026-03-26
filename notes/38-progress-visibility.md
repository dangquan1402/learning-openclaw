# Progress Visibility

> **Question:** How do OpenClaw, GoClaw, DeepAgents, and Claude Code show progress while work is happening?

Progress visibility is one of the highest-value trust layers in agent systems.

## Why it matters

Users trust the system more when they can see:

- what is in progress
- what is blocked
- what already finished
- whether work is still happening

Without this, delegation feels like a black box.

## OpenClaw

OpenClaw has some visibility through:

- session status
- subagent listing and logs
- announce-based completion

This is useful, but still relatively operator-oriented. The user-facing progress layer is lighter than the stronger systems.

### Missing piece

- lightweight todo/progress tracking in the main flow
- clearer active-child visibility before completion

## GoClaw

GoClaw has the heaviest progress model:

- shared task board
- task status lifecycle
- progress updates
- blocker states
- comments
- snapshots and team events

This is the strongest fit for long-running collaborative workflows.

### Limitation

- too heavy for many ordinary assistant requests

## DeepAgents

DeepAgents gives lighter progress signals:

- `write_todos`
- supervisor commentary
- subagent return model

This is strong enough for serious single-user agent work, but it is not a rich workflow board.

### Missing piece

- stronger shared progress model for multi-actor workflows

## Claude Code

Claude Code is very strong here for coding-agent work:

- todo tracking
- teammate task lists
- status changes in the runtime stream

It is lighter than GoClaw, but much more visible than systems that only reveal final answers.

## Best pattern

The strongest general pattern is:

- **todos** for ordinary serious work
- **task boards** only for true team/workflow cases
- **streamed updates** whenever possible

## What OpenClaw should improve

The highest-value improvement is:

- add todo/progress visibility first

That helps:

- single-agent work
- subagent work
- user trust

A full board should come later and only for team workflows.

## Bottom line

Progress visibility is not optional polish. It is part of the control plane.

The best default is:

- lightweight todos for most runs
- heavier boards only when durable coordination is needed
