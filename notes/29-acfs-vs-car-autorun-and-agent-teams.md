# ACFS vs CAR: Autorun and Agent-Team Coordination

> **Parent notes:** [Agentic Coding Flywheel Setup (ACFS)](27-agentic-coding-flywheel-setup.md) | [codex-autorunner (CAR)](28-codex-autorunner.md)
> **Tracking:** [Issue #2](https://github.com/dangquan1402/learning-openclaw/issues/2)

## Short Answer

These two repos are related, but they are solving **different layers**.

- **ACFS** is mainly a **bootstrap + workflow methodology stack**
- **CAR** is mainly a **durable control plane and autorunner**

If the question is "which one is closer to a true autorun/task system?", the answer is **CAR**.

If the question is "which one is stronger for swarm operating method and multi-agent execution discipline?", the answer is **ACFS**.

## Layer Difference

### ACFS

ACFS focuses on:

- environment setup
- planning discipline
- bead-based task decomposition
- swarm rituals using external companion tools

It is best thought of as a **workflow system around a toolchain**.

### CAR

CAR focuses on:

- ticket files as the control plane
- repo-local durable state
- resumable execution
- explicit operator surfaces

It is best thought of as a **task-control product/harness**.

## Direct Comparison

| Axis | ACFS | CAR |
|---|---|---|
| Primary goal | Bootstrap and teach a productive agentic coding environment | Run a durable ticket-driven agent workflow |
| Main abstraction | Plan -> beads -> swarm | Tickets -> runner -> agent execution |
| Shared state | Spread across tools and conventions | Explicit repo-local `.codex-autorunner/` state |
| Human control surface | Shell/tooling/docs-heavy workflow | Web UI + CLI + chat apps |
| Scheduling model | Graph-aware bead routing via `bv` | First incomplete ticket in ascending order |
| Agent communication | Strong via Agent Mail | More task/control-plane centric |
| Beginner story | Very strong | Good, but more system/product oriented |
| VPS/bootstrap story | Core strength | Secondary |
| Durable repo management | Indirect | Core strength |
| Closest match to "autorunner" | Partial | Yes |

## What Each Contributes

### What ACFS contributes

- planning-first workflow
- explicit work units
- swarm coordination habits
- claim/progress/handoff semantics
- graph-aware dependency routing

### What CAR contributes

- durable task state
- repo-local control-plane artifacts
- resume/retry model
- operator surfaces
- structured separation between engine and surfaces

## Recommendation for OpenClaw

For OpenClaw, the highest-value blend is probably:

- **CAR-style durable task/control-plane structure**
- plus **ACFS-style coordination patterns**

I would not copy ACFS's full bootstrap scope unless OpenClaw later decides to own a VPS/developer-environment product.

## Bottom Line

**CAR is the better reference for autorun.**

**ACFS is the better reference for swarm operating method.**

OpenClaw should probably study them on separate tracks first, then combine only the parts that address:

- task state
- execution visibility
- multi-agent coordination
- durable operator control

## Sources

- [ACFS overview](27-agentic-coding-flywheel-setup.md)
- [CAR overview](28-codex-autorunner.md)
