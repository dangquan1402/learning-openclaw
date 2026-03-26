# Four-System Gap Analysis

> **Scope:** OpenClaw, GoClaw, DeepAgents, and Claude Code

Each system is strong in a different layer. The useful question is not "which one wins?" It is:

> what is each one missing, and which missing pieces actually matter?

## The strongest thing in each system

- **OpenClaw**: multi-channel assistant product and local-first gateway
- **GoClaw**: team workflow and durable coordination
- **DeepAgents**: clean supervisor/subagent framework design
- **Claude Code**: strongest overall control plane for coding-agent work

## What each one is missing

## OpenClaw

OpenClaw is missing the most in the **work orchestration layer**:

- weaker todo/progress tracking
- less structured team coordination than GoClaw
- less explicit memory scoping than Claude Code
- weaker eval discipline than DeepAgents

What matters:

- better task/progress visibility
- stronger parent/child result control
- clearer memory separation
- more systematic evaluation

## GoClaw

GoClaw is missing the most in **simplicity and product shape**:

- workflow model is heavy for ordinary assistant tasks
- weaker "default assistant" feel than OpenClaw or Claude Code
- more operational complexity
- license posture reduces openness and reusability

What matters:

- simpler default loop behavior
- better distinction between ordinary delegation and full workflow mode
- less overhead for common requests

## DeepAgents

DeepAgents is missing the most in **product-level collaboration UX**:

- no real shared team board like GoClaw
- less visible human/team workflow support
- fewer built-in product surfaces than OpenClaw or Claude Code

What matters:

- clearer human review surfaces
- more visible multi-agent coordination UX
- stronger out-of-the-box memory/control defaults for non-framework users

## Claude Code

Claude Code is the cleanest overall in control-plane terms, but it is still missing things:

- not built as a multi-channel assistant product like OpenClaw
- less explicit workflow board semantics than GoClaw
- much of the magic is product-controlled rather than fully transparent

What matters:

- better durable task-board semantics if enterprise workflow is the target
- more explicit retrieval-style memory for non-code domains
- more open/extensible architecture if you want to self-host the whole system

## The most important things across all four

After looking across all four, the most important design elements are:

1. **Execution-mode selection**
   The system must know when to stay in the main loop, when to call tools directly, and when to delegate.

2. **Progress visibility**
   Todo tracking or task status matters a lot. Users trust the system more when they can see what is happening.

3. **Parent/child output control**
   Subagent output should not leak raw into the user response. The parent should control synthesis.

4. **Memory layering**
   Scratchpad memory, durable memory, and project instructions should be separated clearly.

5. **Tool and permission scoping**
   Agent capability and approval policy should be separate concerns.

6. **A lightweight default path**
   If every request goes through heavyweight orchestration, the system becomes slow and awkward.

7. **Optional workflow mode**
   Durable boards, dependencies, blockers, and approvals are valuable, but only for the right class of work.

8. **Evals**
   Without evaluation, decomposition and delegation quality are hard to trust.

## What seems genuinely missing in the ecosystem

The bigger gap is not "more subagents." It is a clean combination of:

- Claude Code's control plane
- DeepAgents' framework clarity
- GoClaw's durable workflow layer
- OpenClaw's multi-channel assistant product shape

That combined design is still rare.

## What I think matters most

If the goal is a practical next-generation assistant, the highest-value ingredients are:

- **default loop first**
- **parallel tools for simple independent actions**
- **subagents for heavy isolated branches**
- **todos for most serious work**
- **workflow board only for real team processes**
- **separate memory layers**
- **separate permission layers**
- **strong evals**

## Best synthesis for OpenClaw

The best path for OpenClaw is not to copy any single system.

It should keep:

- OpenClaw's gateway/product identity

And borrow:

- Claude Code's memory/tool/permission control model
- DeepAgents' supervisor/subagent clarity
- GoClaw's workflow ideas only as an optional enterprise mode

## Bottom line

The missing thing is not one feature. It is **balanced layering**:

- lightweight by default
- structured when needed
- durable only where justified

That is the design direction that looks strongest after comparing all four.
