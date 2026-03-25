# Role-Based Agents vs Supervisor Executors

> **Focus:** What changed from the old "specialized agent with its own tools" model to newer supervisor/executor architectures?

## The Confusion

A common first reaction is:

> "Before, each agent had a role, description, and its own tools/skills. Now it looks like the master plans and the executors just do work. Did specialization disappear?"

The answer is **no**. Specialization did not disappear. What changed is that a **coordination layer** was added on top.

## Old Model

The old multi-agent mental model was:

- Agent A = researcher
- Agent B = coder
- Agent C = reviewer
- each has a role
- each may have different tools/skills
- each may talk more directly as an independent actor

This model is still valid.

## Newer Model

The newer pattern adds a **supervisor/master**:

1. the master plans
2. workers execute isolated subtasks
3. the master merges, reviews, or rewrites results
4. the user usually sees only the master's final answer

So the key shift is:

```text
specialized agents only
        ->
specialized agents + orchestration layer
```

## Why Use Executors at All?

The benefit is not only speed.

### Benefits

- **Parallelism** when tasks are independent
- **Context isolation** so each worker sees only one problem
- **Cleaner final output** because the master filters worker noise
- **Failure containment** because one bad branch is isolated
- **Composable specialization** because the master can choose the right worker for each step

### When It Is Not Better

- if the task is tiny
- if all steps depend on each other
- if delegation overhead costs more than direct execution

So yes, multi-agent can be faster, but **the bigger gain is orchestration quality**.

## DeepAgents: What Is Inherited?

DeepAgents supports both:

### A. General-purpose inherited worker

The built-in `general-purpose` subagent is explicitly described as having **all the same capabilities as the main agent**.

Also, in the middleware:

- if a subagent does not specify tools, it inherits the main agent's default tools
- if it does not specify a model, it can inherit the default model
- default middleware can also be inherited

So DeepAgents can absolutely act like a **true inherited executor**.

### B. Specialized worker

A DeepAgents subagent can also define its own:

- name and description
- system prompt
- tools
- model
- middleware
- skills

So in DeepAgents, executors are **not forced** to be generic clones. You can choose between inherited workers and specialized workers.

## GoClaw: What Is Inherited?

GoClaw has two different cases.

### A. `spawn` self-clone executor

This is the real inherited executor path.

From the implementation:

- the subagent inherits the **parent model/provider path** unless overridden
- it builds a tool registry for the subagent from the same system tool stack
- it applies deny-list restrictions and removes recursive spawn/subagent paths
- it builds a subagent system prompt for the child run

So the GoClaw `spawn` tool is basically:

```text
parent agent
  -> cloned child worker
  -> mostly inherited capabilities
  -> same general operating environment
  -> fewer dangerous/recursive paths
```

This is the strongest example of a **self-clone executor** among the systems you asked about.

### B. Team members

GoClaw team members are different.

They are not just inherited clones. They are closer to **real named agents inside a team system**. The lead coordinates them through tasks and mailbox messages.

So in GoClaw:

- `spawn` = inherited self-clone executor
- `team member` = specialized collaborator in a structured team workflow

## Which Spawn/Executor Is Better?

This depends on what you mean by "executor".

### Best true self-clone executor

**GoClaw** is stronger if you mean:

- "spawn a worker that is basically me"
- "inherit my model/provider/tool environment"
- "run it as a child process of my current agent flow"

That is exactly what the current `spawn` path is designed for.

### Best configurable executor framework

**DeepAgents** is stronger if you mean:

- "I want the option of a general inherited worker or a specialized worker"
- "I want to control child prompts, tools, skills, middleware, and models cleanly"
- "I want strong output isolation and framework-level flexibility"

So the answer is:

- **GoClaw** is better for a literal **self-clone executor**
- **DeepAgents** is better for a **designed subagent architecture**

## Final Mental Model

The new architecture should be understood like this:

- The **master** does planning and output control
- The **workers** do isolated execution
- Workers may be:
  - inherited clones
  - specialized agents
  - async remote jobs
  - role-based team members

So no, the new architecture does **not** mean "all executors share the same tools/skills and specialization is gone."

It means:

> specialization still exists, but now it sits inside a stronger orchestration pattern.

## Sources

- [DeepAgents `subagents.py`](https://github.com/langchain-ai/deepagents/blob/main/libs/deepagents/deepagents/middleware/subagents.py)
- [DeepAgents Subagents docs](https://docs.langchain.com/oss/python/deepagents/subagents)
- [GoClaw `subagent_spawn_tool.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_spawn_tool.go)
- [GoClaw `subagent_exec.go`](https://github.com/nextlevelbuilder/goclaw/blob/main/internal/tools/subagent_exec.go)
- [GoClaw Agent Teams docs](https://github.com/nextlevelbuilder/goclaw/blob/main/docs/11-agent-teams.md)
