# codex-autorunner (CAR): Overview

> **Repo:** [Git-on-my-level/codex-autorunner](https://github.com/Git-on-my-level/codex-autorunner)
> **Tracking:** [Issue #3](https://github.com/dangquan1402/learning-openclaw/issues/3)

## What It Is

CAR is a meta-harness for coding agents.

It is designed to let a user run longer, structured implementations using agents they already have, instead of forcing them into one specific model product.

Its key idea is simple:

- **tickets** are the control plane
- **agents** are the execution layer
- the **filesystem** is the source of truth

## Main Goal

CAR is trying to make agent work:

- durable
- inspectable
- resumable
- operator-friendly

It is much closer to a true **autorunner/task-control system** than ACFS.

## Main Architecture

The repo documents four layers:

```text
Engine -> Control Plane -> Adapters -> Surfaces
```

These layers are explicitly enforced by architecture-boundary tests.

That is a strong sign that CAR is not just a collection of scripts. It is a structured software system.

## Core Operating Model

CAR revolves around ticket execution.

The common pattern is:

1. create a plan
2. convert the plan into ticket markdown files
3. store those tickets in the repo state
4. run a flow that picks the next incomplete ticket
5. dispatch that ticket to an agent
6. persist outputs, run metadata, and context documents

The important detail is that tickets are durable files, not just transient chat prompts.

## Repo and Hub Model

CAR supports:

- **hub mode** for managing multiple repositories
- **repo mode** for working directly inside one repository

In hub mode, it keeps central configuration and a manifest of managed repos.

In repo mode, each managed repo gets its own `.codex-autorunner/` area with artifacts such as:

- tickets
- contextspace docs
- run metadata
- configuration/state

## Control-Plane Documents

CAR uses durable context documents, often called **contextspace**, for shared knowledge such as:

- active context
- decisions
- specs

This is important because it keeps long-lived operator/agent context outside the chat window.

## User and Operator Surfaces

CAR supports several control surfaces:

- **web UI**
- **CLI**
- **Telegram**
- **Discord**
- **PMA** (project manager agent)

This makes CAR more product-like than ACFS.

The web UI is presented as the main operator surface, while the CLI remains the most agent-friendly path.

## Execution Environment Model

CAR can run work in different **destinations**:

- local
- Docker

It also documents inheritance rules, doctor checks, and environment contracts for those destinations.

That means CAR is thinking carefully about where agent backends actually run, not just what task they receive.

## What CAR Is Strong At

- durable task control
- repo-local state
- resumable execution
- operator-facing surfaces
- clear separation between execution engine and UI
- filesystem-native artifacts

## What CAR Is Not

CAR is not mainly a VPS bootstrap system or an onboarding wizard.

It is also less focused than ACFS on:

- swarm culture
- explicit claim/handoff rituals
- graph-theoretic work routing

Its center of gravity is the ticket runner and control plane.

## Why It Matters for OpenClaw

CAR is a strong reference for:

- how to represent task state durably
- how to keep shared context inspectable
- how to separate engine, adapters, and surfaces
- how to support long-running agent workflows across repos

It is especially relevant if OpenClaw wants:

- first-class task files or task records
- resume/retry semantics
- operator dashboard surfaces
- chat + UI + CLI coexistence

## Bottom Line

CAR is best treated as an **autorunner and durable control-plane reference**.

For OpenClaw, its main value is in:

- task lifecycle design
- control-plane structure
- persistent operator/agent collaboration artifacts

## Sources

- [CAR README](https://github.com/Git-on-my-level/codex-autorunner)
- [CAR Setup Guide](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/AGENT_SETUP_GUIDE.md)
- [CAR Ticket Skill](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/car-ticket-skill.md)
- [CAR Architecture Boundaries](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/ARCHITECTURE_BOUNDARIES.md)
- [CAR Destinations](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/configuration/destinations.md)
