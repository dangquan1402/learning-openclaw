# Agentic Coding Flywheel Setup (ACFS): Overview

> **Repo:** [Dicklesworthstone/agentic_coding_flywheel_setup](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup)
> **Tracking:** [Issue #4](https://github.com/dangquan1402/learning-openclaw/issues/4)

## What It Is

ACFS is an opinionated system for turning a fresh Ubuntu VPS into a ready-to-use agentic coding environment.

It is not just a shell installer. It combines:

- VPS bootstrap
- developer tooling installation
- multiple coding agent CLIs
- coordination tools
- an explicit workflow philosophy for planning and swarm execution

So ACFS is best understood as both:

- an **environment bootstrap stack**
- and a **workflow/methodology bundle**

## Main Goal

The repo is trying to get a user from:

```text
laptop -> fresh VPS -> configured environment -> running coding agents
```

The beginner story is a major part of the design:

- one-line install
- wizard website
- idempotent setup
- reproducible environment

## Core Architecture

ACFS is organized around a **manifest-driven installer architecture**.

From the repo README and installer:

- `acfs.manifest.yaml` is the central source of truth
- `install.sh` is the main bootstrap entrypoint
- `scripts/lib/*` contains installer and support logic
- `apps/web` holds the wizard website
- `checksums.yaml` and related verification logic support safer upstream installs

The installer itself is strongly operational:

- resumable
- phase/module aware
- preflight capable
- checksum aware
- optimized for unattended setup

## The Workflow Layer

The more interesting part for OpenClaw is not the VPS setup. It is the execution philosophy layered on top.

ACFS separates work into two layers:

### Planning substrate

Use strong frontier models to create and refine a large markdown plan.

### Execution substrate

Use a tool trio to run the project:

- **Agent Mail** for coordination and communication
- **`br`** for dependency-aware task units called beads
- **`bv`** for graph-aware triage of the next best bead

This means ACFS does not center everything in one app. It relies on a set of cooperating tools and a workflow contract.

## The Flywheel Idea

The repo's workflow docs describe a loop like this:

1. create a detailed markdown plan
2. convert the plan into beads
3. use `bv` to choose the highest-leverage ready bead
4. use Agent Mail for claims, reservations, progress, and handoff
5. let a swarm of agents execute the work
6. feed lessons back into the next planning cycle

This is why "Flywheel" is the right name: it is trying to create a compounding workflow, not just run one task.

## What ACFS Is Strong At

- beginner onboarding
- reproducible environment setup
- planning-first execution discipline
- swarm coordination rituals
- dependency-aware task routing
- toolchain composition across many specialized tools

## What ACFS Is Not

ACFS is **not** primarily a repo-local durable task-control system like CAR.

It does not present itself as:

- one canonical ticket runner
- one repo-local state directory for all task control
- one internal task engine with a unified product surface

Instead, it is closer to:

- a bootstrap distribution
- a workflow operating model
- a multi-tool swarm environment

## Why It Matters for OpenClaw

ACFS is useful as a reference for:

- how to separate planning from execution
- how to formalize work units outside ephemeral chat
- how to coordinate multiple agents with claim/handoff rituals
- how an `AGENTS.md`-style operating manual supports long-running swarms

It is less useful as a direct reference for:

- durable ticket state design
- repo-level control-plane storage
- operator dashboard architecture

## Bottom Line

ACFS is best treated as a **workflow and environment reference**.

For OpenClaw, its biggest value is in:

- swarm coordination patterns
- plan-to-task translation
- explicit execution discipline

not in copying the entire bootstrap stack.

## Sources

- [ACFS README](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup)
- [The Flywheel Core Loop](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_CORE_LOOP.md)
- [The Flywheel Approach to Planning and Beads Creation](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/THE_FLYWHEEL_APPROACH_TO_PLANNING_AND_BEADS_CREATION.md)
