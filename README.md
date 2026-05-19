# Task-Scheduler-DAG

A hands-on mini project to learn **DAG-based workflow execution** by building a tiny scheduler from scratch.

## Is DAG basically a workflow?

Short answer: **almost, but not exactly**.

- A **workflow** is the business process (what you want to happen).
- A **DAG** (Directed Acyclic Graph) is one way to model execution dependencies (how steps depend on each other).

So a DAG is a great execution model for many workflows, especially data pipelines and task orchestration.

## Why this project exists

Your concept note describes a production workflow engine with:
- node-by-node execution,
- conditional routing,
- suspend/reenter behavior,
- context persistence.

This project is a simplified learning version that starts from DAG basics and can gradually evolve toward reentrant chatflow-style execution.

## Learning roadmap (build in phases)

### Phase 1 — Static DAG executor

Goal: run tasks in dependency order.

Build:
1. `TaskNode` model (`id`, `name`, `deps`, `handler`).
2. `DAGValidator` to detect cycles.
3. `TopologicalScheduler` (Kahn’s algorithm).
4. `ExecutionContext` to store task outputs.

Outcome: you can define tasks and run them in correct order.

### Phase 2 — Branching and dynamic next nodes

Goal: move from static DAG to workflow-like routing.

Add:
- condition nodes (`if/else`),
- explicit success/failure edges,
- node result object with `status` + `nextNodeIds`.

Outcome: execution path is selected at runtime.

### Phase 3 — Reentrant execution (pause/resume)

Goal: model interactive flow behavior.

Add persisted state:
- `executionId`,
- `currentNodeId`,
- `status` (`RUNNING | WAITING | COMPLETED | FAILED`),
- `context`,
- `waitReason` / `resumePayload`.

Node contract idea:
- `handler()` for first entry,
- `asyncResultHandler()` for resume entry.

Outcome: engine can pause at a node and continue later using same execution ID.

### Phase 4 — Loop/container support

Goal: support foreach/while semantics.

Add:
- loop state stack,
- container node boundaries,
- break/continue semantics.

Outcome: closer to production workflow engines.

## Minimal execution state model

```text
ExecutionState {
  executionId
  currentNodeId
  status
  context
  waitReason
  resumePayload
}
```

## Suggested pseudo-code

```text
while (currentNode != null):
  result = execute(currentNode, context)

  if result.type == SUSPEND:
    save(state)
    return WAITING

  if result.type == FAIL:
    mark_failed(state)
    return FAILED

  save(state)
  currentNode = pick_next(result)

return COMPLETED
```

## How this maps to your note

- Your note’s `nextNodes`/`falsyNodes` maps to runtime routing edges.
- `suspend(...)` + `reenter()` maps to Phase 3’s persisted WAITING state.
- `execution_id` is the stable key for one workflow conversation.

## Practical next step

If you want, next I can scaffold this repo with:
- a tiny Python prototype (`engine.py`, `nodes.py`, `store.py`),
- one sample flow (phone verification style),
- and a CLI demo: `start`, `resume`, `show-state`.
