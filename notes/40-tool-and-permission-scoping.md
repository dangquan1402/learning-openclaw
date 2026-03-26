# Tool and Permission Scoping

> **Question:** How do these systems control what an agent can do, and who decides whether an action is allowed?

This layer is often underspecified, but it is critical.

## Two separate concerns

These should be treated separately:

1. **tool scope**: what an agent is capable of using
2. **permission policy**: when a tool use is allowed or requires approval

Systems get cleaner when they do not mix these two ideas.

## OpenClaw

OpenClaw has strong global tool policy and sandbox controls, especially around session types and non-main sessions.

### Missing piece

- more explicit per-agent capability scoping
- cleaner separation between "agent may use this" and "this call requires approval"

## GoClaw

GoClaw has stronger policy than many frameworks:

- agent identities
- tool groups
- workspace boundaries
- delegation depth restrictions
- team/member role differences

### Missing piece

- simpler, more transparent default mental model for non-operator users

## DeepAgents

DeepAgents is flexible because subagents can have:

- different tools
- different prompts
- different middleware
- different models

This is strong for framework builders.

### Missing piece

- less opinionated permission layer than Claude Code

## Claude Code

Claude Code is strongest here.

It cleanly separates:

- subagent tool inheritance/restriction
- MCP tool access
- permission approvals
- hooks that can allow/deny tool use dynamically

This is one of the most reusable design ideas from the whole comparison set.

## What matters most

The best pattern is:

- per-agent tool scoping
- approval policy as a separate layer
- clear defaults
- visible rules

## What OpenClaw should improve

OpenClaw would benefit most from:

- first-class per-agent tool scoping
- explicit approval modes separate from static tool allow/deny rules
- stronger child-agent capability presets

## Bottom line

Tool scoping and permission policy should not be collapsed into one mechanism.

That separation is one of the clearest signs of a mature control plane.
