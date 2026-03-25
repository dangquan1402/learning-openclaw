# GoClaw: Team Execution and Lead Control

> **Parent:** [GoClaw — OpenClaw Rebuilt in Go](08-goclaw.md)  
> **Repo:** [nextlevelbuilder/goclaw](https://github.com/nextlevelbuilder/goclaw)

## What This Note Focuses On

This note goes deeper on one question: **how does a lead agent in GoClaw actually control worker agents and their output?**

The answer is more specific than "GoClaw has subagents." The current architecture splits the problem into **self-clone subagents** and **team-based delegation**.

## 1. Two Different Mechanisms

### A. `spawn` = self-clone subagent

The `spawn` tool is now for **self-clone subagents only**.

- `mode="async"` returns immediately and the child announces later
- `mode="sync"` blocks and returns the result inline
- the old `"delegate to another named agent"` path was removed
- if the model tries to pass an `agent` parameter, GoClaw explicitly rejects it and tells the model to use `team_tasks(...)` instead

This is an important architectural move: GoClaw no longer mixes "background self-clone" with "team member delegation".

### B. `team_tasks` + `team_message` = real team orchestration

Actual multi-agent teamwork is handled through:

- `team_tasks` for the task board
- `team_message` for the mailbox
- team membership, lead/member roles, and task lifecycle stored in PostgreSQL

So the core idea is:

```text
spawn = clone myself
team_tasks/team_message = coordinate real teammates
```

## 2. Lead-Centric Design

GoClaw is explicitly **lead-centric**.

- the **lead** gets `TEAM.md` orchestration guidance
- members do not get the full orchestration instructions
- members are expected to execute delegated work and report status/results back
- the lead decides task breakdown, sequencing, and final synthesis

This matters because output control is not accidental. The architecture is designed so that the **lead remains the user-facing brain**, while members are execution specialists.

## 3. Task Board as the Control Layer

The task board is the real coordination backbone.

Key task features:

- create, claim, assign, complete, cancel, review, approve, reject
- `blocked_by` task dependencies
- progress updates
- comments, including blocker comments
- attachments and follow-up reminders
- search and list views

This means the lead is not just "calling workers". It is managing a durable work graph with state transitions.

## 4. Parallel Execution

GoClaw supports parallel worker execution through the team system.

The docs describe:

- multiple members working simultaneously
- accumulated sibling results
- async delegation
- real-time team and delegation events over WebSocket

This suggests the intended flow is:

1. lead creates several tasks
2. different members claim or receive them
3. tasks run in parallel
4. partial results are accumulated
5. the lead receives a combined update and decides what to tell the user

## 5. How Output Is Controlled

This is the most important implementation detail.

In `subagent_exec.go`, finished child work is **announced back through the parent agent's session**. The comment in the code is explicit: the announce goes through the parent so the parent agent can **reformulate the result for the user**.

That gives GoClaw a strong output-control model:

- members do work
- the system routes results back to the lead context
- the lead rewrites/synthesizes the result
- only then does the user see the final answer

For team mode, the docs go even further: when sibling delegations finish, their outputs can be **batched** and announced together. That reduces noisy partial delivery and strengthens lead-side synthesis.

## 6. Blocking, Failure, and Escalation

GoClaw also models failure as a first-class team concept.

- members can leave blocker comments
- blocker comments can auto-fail the task
- the lead receives an escalation message
- stale/failed tasks can be retried
- cancelled/rejected tasks can unblock dependents

This is one of the places where GoClaw looks more like a workflow engine than a simple subagent feature.

## 7. Why This Is Stronger Than OpenClaw's Current Story

OpenClaw already has useful subagent orchestration, but GoClaw adds a more explicit control plane:

- durable task state
- lead/member roles
- dependency management
- team mailbox
- review/approval lifecycle
- dedicated UI and event streams

The key upgrade is not just "more subagents". It is **formalized coordination**.

## Bottom Line

GoClaw's team model appears to be designed around a clear rule:

> workers execute, the lead decides what counts, and the lead controls what reaches the user.

That is a better-defined team orchestration story than generic spawn-only systems.

## Sources

- [GoClaw Agent Teams docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
- [GoClaw Tools System docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/03-tools-system.md)
- [GoClaw WS Team Events docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/13-ws-team-events.md)
- [GoClaw `subagent_spawn_tool.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_spawn_tool.go)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
