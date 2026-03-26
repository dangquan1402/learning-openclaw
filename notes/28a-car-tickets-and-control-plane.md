# CAR Tickets and Control Plane

> **Parent:** [codex-autorunner (CAR)](28-codex-autorunner.md)
> **Tracking:** [Issue #3](https://github.com/dangquan1402/learning-openclaw/issues/3)

## Focus

This note zooms in on the part of CAR that matters most for autorun research:

- tickets
- control-plane state
- repo and hub structure
- execution flow

## Core Claim

CAR's core claim is very explicit:

- tickets are the **control plane**
- agents are the **execution layer**
- the filesystem is the **source of truth**

This is the most important lens for understanding the project.

## Ticket Model

CAR uses markdown ticket files with frontmatter.

The docs describe required fields such as:

- `ticket_id`
- `agent`
- `done`

Common optional fields include:

- `title`
- `goal`
- `model`

The ticket body usually includes:

- tasks
- acceptance criteria
- tests
- notes

This means a CAR ticket is not just a title in a task list. It is the main execution contract between the operator and the agent.

## Ticket Sequencing

CAR's default `ticket_flow` is intentionally simple:

- tickets execute in ascending order
- the runner picks the first ticket where `done != true`

That has strong implications:

- prerequisites should be lower-numbered
- tickets should be independently completable at their turn
- reverse dependencies should be avoided

This is less expressive than graph routing, but much easier to reason about.

## Control-Plane Artifacts

CAR keeps durable state under `.codex-autorunner/`.

The exact contents vary by mode, but the design clearly expects durable artifacts such as:

- tickets
- contextspace documents
- run metadata
- configuration
- hub manifest data

This is what makes CAR a control-plane system rather than a loose convention.

## Contextspace

CAR uses shared docs to hold long-lived context for a repo.

Examples from the setup guide:

- `active_context.md`
- `decisions.md`
- `spec.md`

These documents help agents and operators collaborate through files instead of relying only on transient chat context.

## Hub vs Repo

CAR has two main modes:

### Hub mode

A central hub manages multiple repos and provides the main web UI and coordination features.

Typical hub artifacts include:

- a manifest of managed repos
- central configuration
- default repo storage paths

### Repo mode

A repo can also be initialized directly for local CAR usage.

This matters because CAR is designed for both:

- central multi-repo operation
- direct per-repo workflows

## Execution Flow

A simplified CAR flow looks like this:

1. create or import a repo
2. write a plan
3. convert the plan into ticket files
4. store/update shared contextspace docs
5. run the ticket flow
6. CAR selects the next incomplete ticket
7. dispatch that ticket to the configured agent
8. persist outputs and run metadata
9. continue until the ticket set is done

This is a durable, inspectable workflow with explicit artifacts at each step.

## Surfaces and Operators

CAR exposes several ways to work with that control plane:

- web UI
- CLI
- Telegram
- Discord
- PMA

That means the same underlying ticket/control-plane system can be driven through different user surfaces.

This is an important architecture idea:

- **surfaces should not own the truth**
- the control-plane artifacts should remain durable underneath them

## Destinations

CAR also models where work actually runs.

It supports destinations such as:

- local
- Docker

This makes execution environment a first-class part of the configuration, instead of an implicit assumption.

## What Makes This Different

Compared with ACFS, CAR is:

- less about swarm ritual
- more about durable task execution
- less about graph routing
- more about inspectable ticket state

Compared with lighter todo systems, CAR is:

- more structured
- more persistent
- more operator-facing

## Implications for OpenClaw

The strongest reusable ideas here are:

- make task state durable
- keep context in shared files or records
- separate engine from surfaces
- support resume/retry through explicit artifacts
- treat the repo or workspace as a real control-plane boundary

## Bottom Line

CAR's main contribution is not just "run agents on tasks."

Its real contribution is:

- a durable ticket contract
- a repo/hub control-plane model
- clear separation between state, execution, and surfaces

That makes it one of the more relevant references if OpenClaw wants a serious autorun architecture.

## Sources

- [CAR README](https://github.com/Git-on-my-level/codex-autorunner)
- [CAR Setup Guide](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/AGENT_SETUP_GUIDE.md)
- [CAR Ticket Skill](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/car-ticket-skill.md)
- [CAR Architecture Boundaries](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/ARCHITECTURE_BOUNDARIES.md)
- [CAR Destinations](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/configuration/destinations.md)
