# Gem Team: Multi-Agent Orchestration Framework

A modular, high-performance multi-agent team designed for complex project execution, feature implementation, and automated verification.

## 🚀 Overview

Gem Team follows a Strategic Planner/Dynamic Orchestrator pattern. It decomposes high-level user goals into a Directed Acyclic Graph (DAG) of tasks, executes them in parallel across specialized agents, and maintains a rigorous state-controlled workflow.

## 🤖 Agent Roles

| Agent | Model | Specialty | Primary Responsibility |
| :--- | :--- | :--- | :--- |
| `gem-orchestrator` | GLM 4.7 | Coordination | Coordinates multi-agent workflows, delegates tasks, synthesizes results via runSubagent |
| `gem-planner` | Minimax M2.1 | Strategy | Creates DAG-based plans with pre-mortem analysis and task decomposition |
| `gem-implementer` | GLM 4.7 | Execution | Executes TDD code changes, ensures verification, maintains quality |
| `gem-chrome-tester` | Minimax M2.1 | Testing | Automates browser testing, UI/UX validation via Chrome DevTools |
| `gem-devops` | Minimax M2.1 | Infrastructure | Manages containers, CI/CD pipelines, and infrastructure deployment |
| `gem-reviewer` | Minimax M2.1 | Quality | Security gatekeeper for critical tasks—OWASP, secrets, compliance |
| `gem-documentation-writer` | Minimax M2.1 | Knowledge | Generates technical docs, diagrams, maintains code-documentation parity |

## 🔄 Core Workflow

1. **Inception**: The Orchestrator receives a goal and invokes the Planner.
2. **Planning**: The Planner researches the codebase, simulates failure paths (Pre-Mortem), and generates a `plan.yaml` containing 3-7 atomic tasks with dependency mapping.
3. **Plan Approval**: Orchestrator presents plan via `plan_review` and waits for user confirmation (MANDATORY PAUSE).
4. **Delegation**: The Orchestrator identifies "ready" tasks (all dependencies met), applies `parallel_execution`, and launches agents in parallel via `runSubagent` (4-8 concurrent agents).
5. **Execution & Verification**: Workers (Implementer, DevOps, etc.) execute changes and run verification commands before handing back results.
6. **Synthesis**: The Orchestrator processes handoffs, updates `plan.yaml`, spawns documentation-writer if needed, triggers review, and routes tasks.
7. **Batch Confirmation**: Orchestrator presents batch summary via `walkthrough_review` and waits for user confirmation (MANDATORY PAUSE).
8. **Loop**: Repeat steps 4-7 until all tasks complete.
9. **Delivery**: Results are presented via a comprehensive `walkthrough_review` summary.

## 🛠 Key Features

### ⚡ Parallel Execution Engine

The system supports parallel execution with dynamic batching:

- **Heavy tasks** (implementation, review): max 4 concurrent agents
- **Lightweight tasks** (lint, format, typecheck): max 8 concurrent agents

### 🧠 Parallel Execution

The Orchestrator applies intelligent task splitting for `lint|format|typecheck|refactor|cleanup` tasks:

**Strategy Detection:**

- Use `task.parallel_strategy` from plan.yaml if set: `none|by_directory|by_file|by_module`
- Fallback to pattern matching:
  - `lint|format|test`: by directory, max 8 slots
  - `typecheck`: by file, max 8 slots
  - `refactor`: by module, max 4 slots
  - `verify`: parallel only, max 4 slots

**Expansion:**

- Consolidate overlapping file operations
- Create ephemeral sub-tasks: `{task_id}@{split_key}`
- Track in memory (NOT plan.yaml)

**Smart Batching:**

- Batch 1: All lint|format sub-tasks
- Batch 2: All typecheck sub-tasks
- Batch 3: Other refactors

**Lazy Verification:**

- Sub-tasks perform quick syntax checks only
- Full verification (full lint, tests) deferred to parent task after all sub-tasks complete

### 🛡 Verification-First

No task is considered complete without passing its defined `verification` command. Implementers are required to check `get_errors` after every edit to ensure zero regression.

### 📝 Plan Continuity

State is persisted in `docs/.tmp/{plan_id}/plan.yaml`. This allows the team to recover from interruptions, handle complex retries, and provide a clear audit trail of the project's evolution.

### 🔒 Agent Hierarchy

```text
User → Orchestrator → Subagents (via runSubagent)
```

- **Orchestrator**: `disable-model-invocation: true` - delegates via `runSubagent`, never executes tasks directly
- **Subagents**: `disable-model-invocation: false` - execute tasks via tools
- **Subagents CANNOT call other subagents** - all cross-agent collaboration mediated by orchestrator

## 📁 Project Structure

- `gem-*.agent.md`: Concise agent definitions with role, expertise, mission, workflow, and operating rules.
- `plan.yaml`: The single source of truth for the current project state.
- `README.md`: This file.

---
*Built for Gem Team — Precision. Parallelism. Progress.*
