# CAR PMA and Chat Surfaces

> **Parent:** [codex-autorunner (CAR)](28-codex-autorunner.md)
> **Tracking:** [Issue #3](https://github.com/dangquan1402/learning-openclaw/issues/3)

## Focus

This note covers the more operator-facing side of CAR:

- PMA
- web UI
- Telegram
- shared chat behavior
- durable surface state

## PMA as a Management Layer

CAR includes a **Project Management Agent (PMA)**.

The important point is that PMA is not the same thing as the normal ticket runner.

PMA is a conversational management layer that helps the operator:

- create or edit tickets
- manage repos/worktrees
- respond to dispatches
- handle project setup
- move files through the system

So PMA is effectively a control operator interface implemented as an agent.

## PMA Durable State

The PMA runbook shows that PMA has durable hub-scoped artifacts under `.codex-autorunner/`.

Examples include:

- `filebox/inbox/`
- `filebox/outbox/`
- `pma/AGENTS.md`
- `pma/active_context.md`
- `pma/context_log.md`

This is important because PMA is not only chat history. It has durable workspace files and durable operational docs.

## File Transfer Model

PMA supports file movement through explicit inbox/outbox directories.

The documented model is:

- user uploads into PMA inbox
- agent reads from inbox
- agent writes outputs into outbox
- user downloads from outbox

That is a practical design for mobile or remote chat surfaces where direct workspace access is weak.

## Surface Principle

CAR's architecture docs emphasize that surfaces should not own the truth.

The surfaces:

- web UI
- Telegram
- Discord
- CLI

all sit on top of the same underlying durable control-plane artifacts.

This is a strong architectural choice because it means chat integrations do not become the only source of state.

## Telegram as an Interactive Surface

The Telegram guide makes clear that CAR treats Telegram as a real interactive surface, not just notifications.

It supports:

- commands
- normal chat turns
- repo binding
- PMA mode
- topic-aware collaboration policy
- active/command-only/silent chat modes

This is more mature than a simple "send messages to a bot" pattern.

## Collaboration Policy

One of the more notable ideas is explicit collaboration policy for shared chats.

The docs describe modes such as:

- `active`
- `command_only`
- `silent`

and support topic-aware routing for Telegram supergroups.

This matters because shared human/agent chat spaces often fail when everything is treated as one global active channel.

CAR is trying to solve that by giving operators explicit per-destination behavior.

## Binding and Context

The Telegram workflow includes concepts like:

- `/bind <repo_id|path>`
- `/pma on`
- `/status`
- `/ids`

That means the chat surface is not free-floating. It is bound into the repo/hub control plane with durable context and routing rules.

## Operator Experience

From the docs, CAR is aiming for a workflow where an operator can:

- manage tickets from the web UI
- talk to the PMA from chat
- upload or retrieve files through PMA
- use Telegram or Discord without losing durable state
- keep repo execution tied to hub/repo artifacts

This makes CAR feel much more like a multi-surface operations product than a plain CLI wrapper.

## Why It Matters for OpenClaw

The strongest reusable ideas here are:

- keep chat surfaces thin and durable-state-backed
- separate conversational management from raw task execution
- support explicit modes for shared chats
- give operators file inbox/outbox patterns when direct filesystem access is weak
- preserve surface-independent context artifacts

This is especially relevant if OpenClaw wants to support:

- Telegram or Discord operators
- mobile or lightweight operator workflows
- human/agent collaboration outside the terminal

## Bottom Line

CAR's PMA and chat surfaces are not just add-ons.

They are part of a broader design principle:

- keep state durable
- let many surfaces drive the same system
- avoid making chat the only source of truth

That is one of the more practically useful ideas CAR offers for OpenClaw.

## Sources

- [CAR README](https://github.com/Git-on-my-level/codex-autorunner)
- [CAR Telegram Setup Guide](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/AGENT_SETUP_TELEGRAM_GUIDE.md)
- [PMA File Transfer E2E Verification Runbook](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/pma-file-transfer-e2e-runbook.md)
- [CAR Architecture Map](https://github.com/Git-on-my-level/codex-autorunner/blob/main/docs/car_constitution/20_ARCHITECTURE_MAP.md)
