# Yesod — SwarmCraft (Execution Engine / Foundation)

## Summary
SwarmCraft is the execution engine that turns plans into real outputs. It receives structured tasks from upstream planning/governance, dispatches them to agents (AI services, tools, humans), enforces dependencies and constraints, and returns deterministic status/results plus full telemetry.

SwarmCraft is an **execution archetype**: any concrete orchestrator implementation (e.g., a matrix-driven engine) is an instance unless the entire ecosystem standardizes it.

---

## Responsibilities
- Execute upstream-defined work faithfully (tasks are not reinterpreted).
- Dispatch tasks to the appropriate executor class (AI service, tool adapter, human workflow).
- Manage ordering, dependencies, and safe concurrency.
- Enforce Orgo constraints embedded in tasks (timeouts, budgets, deadlines, resource caps).
- Produce concrete outputs/deliverables and register their locations/identifiers.
- Emit complete execution telemetry: lifecycle events, correlation IDs, error codes, output metadata.

---

## Contract Boundaries

### Upstream
- Producer: Architect (planning) and/or Orgo (governed task lifecycle).
- Input: a **structured task contract** with fields such as `task_id`, `agent/role`, `task_type`, `inputs/context`, `output_target`.

SwarmCraft does not decide *what* should be done; it decides *how and when to execute what it is told*, within constraints.

### Downstream
- Consumers: file system, repositories, deployment targets, data transforms, UI surfaces, external APIs, human contributors.
- Output: artifacts/outcomes (documents, code, datasets, deployments) + execution logs and status reports.

### Authority limits
- SwarmCraft MUST NOT mutate canonical truth (Kristal Exchange) directly. If execution generates new knowledge, it must re-enter the governed pipeline via Orgo as new work.

---

## Inputs
Minimum required:
- `TaskContract` (typed object)
- Context references (e.g., Kristal IDs / pack references / input files / prior outputs)
- Constraints (deadlines, budgets, allowed tools, environment)

Optional:
- Human routing metadata (assignee pools, escalation rules)
- Quality gates to run as sub-tasks (tests, lint, review)

---

## Outputs
- `TaskResult` (typed):
  - `status` (SUCCESS | FAILED | BLOCKED | NEEDS_INPUT | CANCELLED)
  - `output_refs` (paths, URIs, content IDs, build artifacts)
  - `error_code` and diagnostic payload (if failed)
  - `metrics` (duration, retries, resource usage)
- Execution telemetry stream (see Observability)
- Optional: “completion triggers” signaling Orgo/Architect that a plan phase is done.

---

## Primary Artifacts

### 1) Task Contract (boundary object)
**Intent:** encode execution intent without ambiguity.

Recommended minimal fields:
- `task_id` (stable identifier)
- `task_type` (enum)
- `agent_role` (executor class)
- `inputs` (typed refs; no hidden dependencies)
- `output_target` (where output must go + format expectations)
- `constraints` (timeout, budget, deadline, environment)
- `correlation`:
  - `case_id` / `workflow_id` / `build_id` (when present)
  - `origin_node` (Architect/Orgo)
  - `mandate_id` (recommended propagation)

### 2) Task Execution Log (telemetry-grade)
Each task emits:
- start/end timestamps
- agent ID / worker ID
- status transitions
- error codes (stable)
- output metadata and locations
- correlation IDs (task_id, workflow/build ids)

### 3) Execution Report (plan-level)
Aggregates per-task results into:
- plan completion status
- partial execution summary (what completed vs blocked)
- quality gate outcomes
- resource usage summary

---

## Invariants (hard)
1) **Faithful execution**
   - SwarmCraft must not change task intent; if something is wrong, it fails/blocks and reports.
2) **Atomic outcomes per task**
   - Each task completes with a defined result or fails with a clear error; partial states must be internally contained and not exposed as “success”.
3) **Dependency correctness**
   - If Task B depends on Task A, SwarmCraft must enforce ordering; independent tasks may run in parallel safely.
4) **Constraint compliance**
   - Timeouts, budgets, and deadlines must be enforced when declared.
5) **Full telemetry**
   - Every task must emit lifecycle events and correlate back to the originating request/build/workflow.

---

## Scheduling and Concurrency
SwarmCraft may parallelize independent tasks (“swarm”) but must:
- respect declared dependencies
- avoid resource conflicts
- ensure deterministic integration/merge ordering when multiple workers produce outputs for the same deliverable

Recommended mechanism:
- DAG scheduling with explicit dependency edges
- bounded concurrency per resource class (CPU, API quota, human bandwidth)
- deterministic merge policies (e.g., stable ordering by task_id)

---

## Quality Control (execution-level)
Execution quality is “does the output meet the task spec”, not “is the knowledge true” (truth is governed earlier).

SwarmCraft may enforce:
- automated checks as tasks (tests, lint, schema validation, diff rules)
- human review stages for critical outputs
- failure behavior: mark task incomplete and request upstream adjustment rather than fabricate missing pieces

---

## Human-in-the-loop
SwarmCraft can route tasks to humans and must:
- handle waiting/timeout/escalation
- preserve accountability (who did what, when)
- capture review outcomes as structured results (pass/fail + notes)

---

## Failure Modes (recommended stable codes)
- `TASK_CONTRACT_INVALID` — schema/required fields missing
- `INPUT_MISSING` — required files/refs not present
- `EXECUTOR_UNAVAILABLE` — agent role not available
- `CONSTRAINT_VIOLATION` — budget/deadline/timeout exceeded
- `DEPENDENCY_FAILED` — upstream prerequisite failed
- `OUTPUT_SPEC_MISMATCH` — output produced but does not match required format/target
- `EXTERNAL_API_ERROR` — downstream service/tool failure
- `HUMAN_TIMEOUT` — assigned human did not respond in time

All failures must be reported deterministically with correlation IDs.

---

## Observability
Minimum telemetry:
- Task lifecycle events: DISPATCHED, STARTED, RETRIED, COMPLETED, FAILED, BLOCKED
- `task_id`, `agent_id`, `workflow_id`/`build_id` when present
- timestamps and durations
- error codes + diagnostics
- output refs and basic metrics (size, counts, test pass rates)

Recommended metrics:
- success/failure rate by task_type and agent_role
- retry distributions
- p50/p95 latency per executor class
- resource usage per workflow
- deadline miss rate

---

## Interfaces (conceptual)
- `POST /swarm/tasks:enqueue` (accepts TaskContract)
- `GET /swarm/tasks/{task_id}` (status + result refs)
- `GET /swarm/workflows/{workflow_id}` (aggregate execution report)
- `SUBSCRIBE /swarm/events` (telemetry stream)

Note: interfaces are illustrative; concrete transports may be queue-based (pub/sub, job queues) rather than HTTP.

---

## Conformance Tests (minimum)
1) Invalid TaskContract → reject with `TASK_CONTRACT_INVALID` (no execution).
2) Dependency enforcement → B never starts before A success.
3) Atomicity → no “half-success”; failures emit stable error codes and no partial activation of deliverables.
4) Constraint enforcement → timeouts/budgets stop execution and produce deterministic error.
5) Telemetry completeness → every task emits start/end and correlation IDs.
6) “No truth mutation” → execution outputs never directly alter Kristal Exchange; any knowledge updates become governed work.

---

## Example Instance (informative): Matrix-driven orchestrator pattern
A matrix-driven engine can implement SwarmCraft by:
- reading a single project state (“matrix”) as truth for current status
- dispatching tasks to specialized services
- applying a strict separation between logic (orchestrator), stateless AI services, and passive data stores

This is an instance pattern, not a universal requirement of the ecosystem.

---

## Open Questions / TODO
- Standardize `TaskContract` schema (shared across Orgo ↔ SwarmCraft).
- Define stable error code enums and diagnostic payload contract.
- Define deterministic merge strategies for multi-agent outputs (per artifact type).
- Define minimum telemetry retention policy and privacy constraints.
