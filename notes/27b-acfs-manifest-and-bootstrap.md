# ACFS Manifest and Bootstrap Architecture

> **Parent:** [Agentic Coding Flywheel Setup (ACFS)](27-agentic-coding-flywheel-setup.md)
> **Tracking:** [Issue #4](https://github.com/dangquan1402/learning-openclaw/issues/4)

## Focus

This note covers the part of ACFS that is less about swarm method and more about system packaging:

- manifest-driven installer design
- generated scripts
- bootstrap behavior
- environment opinionation

## Single Source of Truth

ACFS is explicit that the installer stack is manifest-driven.

The maintainer guide describes the flow like this:

```text
acfs.manifest.yaml
  -> generator
  -> scripts/generated/
  -> install.sh orchestrator
```

That means the manifest is not just documentation. It is the canonical definition of:

- modules
- phases
- dependencies
- install commands
- verification rules

## Manifest Shape

The manifest defines modules with fields such as:

- `id`
- `description`
- `category`
- `phase`
- `run_as`
- `optional`
- `enabled_by_default`
- `dependencies`
- `installed_check`
- `install`
- `verify`
- `tags`

This is effectively a package graph for the ACFS environment.

## Module Phases

The manifest uses phased execution.

Examples visible near the top:

- base system packages
- target-user normalization
- filesystem setup
- shell setup
- CLI tools

That gives ACFS a more structured bootstrap process than a long linear shell script.

## Orchestrator vs Generated Logic

The repo separates:

- **manifest** as definition
- **generator** as translation layer
- **generated bash** as executable module logic
- **install.sh** as orchestration layer

This matters because it keeps the installer maintainable as the tool list grows.

Without this split, ACFS would likely collapse into a very large hand-maintained shell script.

## Bootstrap Behavior

The installer is designed for high automation:

- resumable
- idempotent
- module-aware
- phase-aware
- preflight-aware
- capable of running in unattended mode

The `install.sh` header also shows support for:

- `--resume`
- `--force-reinstall`
- `--skip-*`
- `--only`
- `--only-phase`
- `--print-plan`
- `--list-modules`

That makes ACFS closer to a real environment orchestrator than a one-shot setup script.

## Security and Verification

The maintainer guide documents a strict model for external installers:

- HTTPS only
- checksum verification
- known installer allowlisting

This is backed by:

- `checksums.yaml`
- verified installer logic
- security helper scripts

That is important because ACFS is intentionally doing a lot of network bootstrap work.

## Web Generation

An unusual but important detail is that the same manifest also drives website content.

The guide says manifest metadata can generate web-facing TypeScript files for:

- tool cards
- TL;DR summaries
- command references
- lesson navigation

So ACFS uses the manifest for both:

- install/runtime definition
- content generation for the onboarding website

## Environment Opinionation

The manifest and installer clearly encode a strong opinion about the target environment:

- Ubuntu VPS
- passwordless sudo in vibe mode
- `/data/projects` workspace layout
- installed `AGENTS.md` in workspace root
- zsh and shell UX defaults
- multiple agent CLIs and stack tools

This is not a neutral installer. It is a curated agentic coding distribution.

## Why It Matters for OpenClaw

The most reusable ideas here are:

- use a manifest as the source of truth when environment setup grows complex
- separate definition, generation, and orchestration
- include verification and drift checks
- keep bootstrap state resumable and phase-aware

The less reusable part is the full environment opinion:

- OS assumptions
- filesystem conventions
- stack bundling
- one-click VPS philosophy

## Bottom Line

ACFS's bootstrap architecture is one of its real strengths.

It shows how to scale an opinionated environment setup without turning the project into an unmaintainable shell script.

For OpenClaw, the main lesson is:

- copy the **manifest-driven packaging discipline**
- not necessarily the full VPS/bootstrap product scope

## Sources

- [ACFS README](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup)
- [ACFS Maintainer Guide](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/docs/MAINTAINER_GUIDE.md)
- [ACFS Manifest](https://github.com/Dicklesworthstone/agentic_coding_flywheel_setup/blob/main/acfs.manifest.yaml)
