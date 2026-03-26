# Prompt vs Memory vs Runtime State

> **Question:** What should live in the system prompt, what should live in memory, and what should live in runtime state?

This distinction is one of the most important architecture decisions in agent systems.

## The three layers

## 1. System prompt

This should contain:

- stable behavioral rules
- tool-usage guidance
- safety and permission expectations
- short durable operating instructions

Good examples:

- "use tools directly"
- "ask before risky actions"
- "subagent output is internal"

Bad examples:

- huge volatile git status dumps
- long temporary task context
- transient blocked states

## 2. Memory

This should contain:

- durable facts
- user preferences
- project conventions
- architectural decisions
- reusable lessons

Good examples:

- important commands
- where core files live
- coding conventions
- stable user preferences

Bad examples:

- current task status
- temporary debugging notes
- one-off execution traces

## 3. Runtime state

This should contain:

- current task list
- active subagents
- pending approvals
- progress updates
- temporary execution context

Good examples:

- todo list for this run
- "waiting on user approval"
- active child run IDs
- current workflow/task statuses

Bad examples:

- evergreen project instructions
- long-term architectural memory

## How the four systems differ

## OpenClaw

OpenClaw is relatively balanced:

- compact section-built prompt
- workspace/bootstrap files for durable context
- session/subagent state outside the prompt

Its main risk is that bootstrap injection can still blur prompt and durable memory if too much is loaded every turn.

## GoClaw

GoClaw pushes a lot into the prompt builder:

- tooling
- safety
- persona
- team context
- context files
- recency reminders

But it also has strong external memory and runtime state.

Its biggest risk is prompt heaviness when too much generated context is injected.

## DeepAgents

DeepAgents is the clearest separation:

- base prompt stays smaller
- memory lives in middleware/backends
- execution state lives in graph/runtime state

This is one of its strongest design qualities.

## Claude Code

Claude Code pushes the most into the system layer:

- behavior policy
- tool policy
- risky-action rules
- auto memory instructions
- `MEMORY.md`
- environment snapshot
- MCP instructions
- git status snapshot

This gives it a very strong operating contract, but also makes the system payload heavier and more volatile.

## Best rule of thumb

The cleanest architecture is:

- **system prompt** for stable rules
- **memory** for durable facts
- **runtime state** for live execution state

If these are mixed too much:

- prompts get bloated
- memory becomes noisy
- runtime control becomes harder to reason about

## What OpenClaw should do

The best direction for OpenClaw is:

- keep a compact prompt
- keep durable instructions in curated files
- avoid injecting too much volatile state into the prompt
- make progress/approvals/tasks explicit runtime state

## Bottom line

Many systems struggle not because they lack features, but because they put the right information in the wrong layer.

This separation is one of the clearest markers of architectural maturity.
