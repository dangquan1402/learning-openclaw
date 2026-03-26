# Parent-Child Output Control

> **Question:** How do these systems control what a child agent returns, and who is allowed to speak to the user?

This layer matters because raw worker output is often noisy, partial, or unsafe to show directly.

## Core principle

The strongest systems treat child output as:

- internal work product
- not final user-facing text

The parent should own the final response.

## OpenClaw

OpenClaw already follows this pattern fairly well.

Subagents announce results back to the parent/requester flow, and the parent is expected to rewrite or synthesize the result in normal assistant voice.

### Missing piece

- clearer result envelopes
- better synthesis policies
- stronger visibility into active child/result state

## GoClaw

GoClaw is strong here in both modes:

- self-clone spawn announces through the parent
- team members return work to the lead
- the lead synthesizes the final user-facing result

This is one of GoClaw's best architectural choices.

## DeepAgents

DeepAgents is one of the cleanest examples of output quarantine:

- subagents do heavy work in isolation
- the parent receives a final tool result
- the user does not directly see the subagent's internal trace

This keeps context cleaner and gives the parent strong control.

## Claude Code

Claude Code is also built around parent-owned output:

- subagents work in separate context
- the main session receives the result
- the main session answers the user

This is one reason Claude Code feels coherent even when delegation happens.

## What matters most

The important rules are:

- child agents should not usually speak directly to the user
- child results should be compact and structured
- the parent should decide what becomes visible

## What OpenClaw should improve

OpenClaw already has the right direction. The next improvements should be:

- standardized child result schema
- stronger parent-side summarization rules
- better progress/result dashboards for active children

## Bottom line

Parent/child output control is one of the most important architecture layers.

If delegation is added without this, the system becomes noisy and hard to trust.
