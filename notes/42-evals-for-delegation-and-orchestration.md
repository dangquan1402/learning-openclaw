# Evals for Delegation and Orchestration

> **Question:** What should be evaluated if you want to trust planning, delegation, and multi-agent execution?

This is one of the biggest gaps in many agent systems.

## Why evals matter

Without evaluation, it is hard to know whether:

- the system delegated when it should
- it stayed simple when it should
- child results were handled correctly
- memory helped or hurt
- workflow mode improved the outcome

## What to measure

The highest-value eval areas are:

1. **execution-mode selection**
   Did the system choose main loop, tools, subagents, or workflow mode appropriately?

2. **task decomposition**
   Did it split work into sensible branches?

3. **child result handling**
   Did the parent synthesize correctly without leaking raw internal output?

4. **memory behavior**
   Did the system recall the right information at the right time?

5. **progress visibility**
   Were todos or task states useful and accurate?

6. **cost and latency**
   Did orchestration improve outcomes enough to justify extra overhead?

7. **workflow correctness**
   For board-based systems: were blockers, dependencies, retries, and approvals handled correctly?

## System comparison

- **DeepAgents** is strongest here conceptually and operationally
- **Claude Code** likely has strong internal eval discipline, but less open evaluator visibility
- **GoClaw** has rich workflow state that is evaluable, but less obvious eval maturity
- **OpenClaw** would benefit the most from a stronger published eval layer

## What OpenClaw should do

OpenClaw should add targeted evals for:

- when to spawn
- when not to spawn
- announce/result quality
- memory recall quality
- follow-up coherence after delegation

These are more important than generic benchmark scores.

## Bottom line

If a system delegates, it should evaluate delegation.

Evals are part of the architecture, not just a testing afterthought.
