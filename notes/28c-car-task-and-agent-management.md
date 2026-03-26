# CAR Task and Agent Management

> **Parent:** [codex-autorunner (CAR)](28-codex-autorunner.md)
> **Tracking:** [Issue #3](https://github.com/dangquan1402/learning-openclaw/issues/3)

## Focus

This note ignores installation details and focuses only on how CAR manages:

- task state
- run state
- agent dispatch
- human handoff
- multi-surface control

## Core Model

CAR treats task and agent management as a durable control-plane problem.

Its main components are:

- ticket files for task intent
- flow state for execution history and status
- repo-local and hub-local durable stores
- dispatch/reply for agent-human interaction

This is a more explicit system-of-record approach than ACFS.

## Task Unit: Tickets

The core task abstraction is the **ticket**.

Tickets are markdown files with frontmatter and are the main control-plane artifact.

Important characteristics:

- durable on disk
- independently editable
- agent-assigned
- testable via acceptance criteria

CAR therefore manages tasks through explicit documents rather than through chat memory or a mostly implicit work queue.

## Task Ordering

CAR's documented `ticket_flow` is intentionally simple:

- execute tickets in ascending order
- pick the first ticket where `done != true`

This means CAR is much more queue-like than ACFS.

Its strength is not graph routing. Its strength is predictable, durable sequencing.

## Canonical Run State

The run-history docs are explicit that canonical run state lives in:

- `.codex-autorunner/flows.db`

This store tracks:

- flow runs
- events/timeline
- artifacts

The important point is that run history is not supposed to be reconstructed by scanning loose files first. CAR defines a canonical store.

## Canonical State Roots

The state-roots contract is also very explicit.

CAR divides durable state across:

- repo-local `.codex-autorunner/`
- hub-local `.codex-autorunner/`
- global `~/.codex-autorunner/`

It also defines authority boundaries for:

- `flows.db`
- orchestration SQLite
- transport databases
- PMA artifacts

This is a strong sign that CAR cares deeply about who owns each kind of state.

## Agent Dispatch and Human Handoff

The architecture map describes a dispatch model with:

- **Dispatch**: agent -> human
- **Reply**: human -> agent

Dispatch can be:

- `notify`
- `pause`

This is significant because it formalizes when an agent can keep going and when it must stop and wait for operator input.

That is more structured than simple chat interruption.

## Multi-Surface Control

CAR also supports several surfaces without letting any one surface own the truth:

- web UI
- CLI
- Telegram
- Discord
- PMA

Task and run state stay durable underneath these surfaces.

That means an operator can switch surfaces without losing the control plane.

## Process Management

CAR also keeps durable metadata for long-lived subprocesses under `.codex-autorunner/processes/`.

This registry exists for:

- auditability
- crash recovery
- stale-process cleanup
- correlating CAR processes with child agent/app-server processes

So CAR manages agents not only as logical tasks, but also as managed runtime processes.

## What This Means Architecturally

CAR task and agent management is best described as:

- **durable ticket queue**
- **canonical run/event store**
- **formal dispatch/reply handoff**
- **multi-surface operator access**
- **managed subprocess registry**

It is less swarm-like than ACFS, but more explicit about system-of-record state.

## Relevance to OpenClaw

If OpenClaw wants stronger task/agent management, CAR suggests:

- make task state canonical and durable
- define who owns run history and artifacts
- support human handoff as a first-class state transition
- keep chat/UI layers thin over the same underlying records
- treat long-lived agent processes as explicit managed resources

## Bottom Line

CAR manages tasks and agents through a **control-plane architecture**, not mainly through swarm habits.

The key idea is:

- represent tasks, runs, artifacts, and handoffs in durable canonical stores
- let surfaces and agents operate on top of that shared state

## Sources

- [CAR README](https://github.com/Git-on-my-level/codex-autorunner)
- [Run History Contract](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/RUN_HISTORY.md)
- [State Roots Contract](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/STATE_ROOTS.md)
- [Architecture Map](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/car_constitution/20_ARCHITECTURE_MAP.md)
- [Managed Process Registry](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/process-registry.md)
