# Execution Path (Protocol) — Plan → Tasks → Execution → Telemetry

This document specifies the **Execution Path**: how action moves through the ecosystem as governed work. It defines the protocol boundary from **Architect-Strategy** to **Orgo**, from **Orgo** to **SwarmCraft**, and back through **telemetry** and **governed feedback**.

The Execution Path is about **doing** (work and state changes), not about **declaring truth** (canon). Canon is owned by Kristal.

---

## 1) Purpose

The Execution Path exists to:
- convert intentions (plans) into **governed, auditable, executable tasks**
- enforce **policy gates** before action
- ensure execution is **observable** and **replayable at the audit level**
- route outcomes back into governance without mutating canon

---

## 2) Actors (nodes) and responsibilities

- **Architect-Strategy (Netzach)**: produces Plan Envelope and proposes Task Graph.
- **Orgo (Gevurah)**: validates, normalizes, routes, and gates tasks; owns the lifecycle of Cases/Tasks.
- **SwarmCraft (Yesod)**: executes tasks via agents/tools/humans; emits telemetry; produces artifacts.
- **Konnaxion (Chesed)** (optional surface): exposes execution status/notifications; collects collaboration signals.
- **EkoH (Malkuth)** (optional): aggregates trust/impact signals (never mutates canon).

---

## 3) Core invariants (must always hold)

1) **Governance before execution**: nothing executes unless it is an Orgo Task in an executable state.
2) **Pinned context**: tasks must reference pinned mandate/blueprint and (when relevant) pinned exchange/pack IDs.
3) **Auditability**: execution must emit telemetry with correlation IDs and artifact lineage.
4) **No canon mutation**: execution outcomes may create new artifacts and new work, but cannot mutate Kristal Exchange directly.
5) **Deterministic gating**: Orgo’s gate decisions must be reproducible given the same task + policy versions.

---

## 4) Artifacts (contracts moved along this path)

### 4.1 Plan Envelope (Architect-Strategy → Orgo)
- references: `mandate_id`, `blueprint_version`, `source_exchange_id` and/or `pack_id` (pinned)
- includes: objectives, constraints, assumptions, task graph proposal, justification refs

### 4.2 Orgo Case + Task (Orgo → SwarmCraft)
- **Case**: long-lived container
- **Task**: atomic unit of work (the executable contract)
- includes: routing, priority, method, inputs, expected outputs, constraints, telemetry contract, lineage

### 4.3 Telemetry Events (SwarmCraft → Orgo)
Minimum event types:
- `TASK_STARTED`
- `STEP_STARTED`
- `STEP_FINISHED`
- `TASK_FINISHED`
- `TASK_FAILED`
- `ARTIFACT_PRODUCED`

### 4.4 Execution Result (SwarmCraft → Orgo)
- task status transitions
- produced artifacts (IDs, types, hashes)
- deterministic error codes on failure
- operator/human approvals (if required)

---

## 5) Protocol sequence (canonical)

### 5.1 Plan submission
1) Architect-Strategy emits a **Plan Envelope** + proposed Task Graph.
2) Orgo receives and stores the plan proposal (linked to mandate/blueprint/canon references).

### 5.2 Task normalization and gating (Orgo)
Orgo MUST:
- validate schema of Tasks
- enforce mandate + blueprint constraints
- enforce capability constraints (tools allowed/forbidden, required approvals)
- ensure pinned references (no floating “latest”)
- assign routing + visibility + ownership defaults
- set task status to `DRAFT` or `READY` (never directly to RUNNING)

If any gate fails:
- Tasks remain `BLOCKED` or `DRAFT` with structured reasons
- Orgo may open a “blocked” Case or request clarification
- no execution occurs

### 5.3 Dispatch to SwarmCraft
When Task is `READY` and scheduled:
- Orgo dispatches the Task contract to SwarmCraft
- Orgo provides:
  - `task_id`, `case_id`, `tenant_id`
  - inputs (artifact refs)
  - method and parameters
  - constraints and approvals required
  - telemetry contract and correlation rules

### 5.4 Execution
SwarmCraft MUST:
- emit `TASK_STARTED` with `task_id` and correlation id
- run steps (agent/tool/human) according to `method`
- emit `STEP_*` events and attach step-level diagnostics
- produce declared output artifacts (or fail deterministically)
- emit `ARTIFACT_PRODUCED` for each produced artifact (with type + id + hash)
- finish with `TASK_FINISHED` or `TASK_FAILED`

### 5.5 Status reconciliation
Orgo ingests telemetry and MUST:
- reconcile task status transitions
- attach produced artifacts to the task record
- compute outcome category (success, failure, partial, needs-review)
- enforce postconditions (e.g., human approval required before marking DONE)

### 5.6 Feedback loop (governed)
If execution produces:
- contradictions
- errors
- new evidence
- user feedback

…Orgo MUST create new governed work (Cases/Tasks) rather than mutating canon in place.

---

## 6) Task contract requirements (minimum)

A Task is executable only if it includes:
- identity: `task_id`, `tenant_id`, `case_id`
- routing: base/category/subcategory (+ optional horizontal role/tags)
- priority: P0–P4
- method: agent/tool/human/hybrid (+ parameters)
- inputs: artifact refs (optional but recommended)
- expected outputs: at least one expected output
- constraints: deadline/max attempts/approval flags/tool allowlists
- telemetry contract: correlation requirement + required events + required fields
- lineage: plan_id and pinned exchange/pack refs when relevant

---

## 7) Gate rules (normative)

### 7.1 “Pinned references” rule
If a task references canon/runtime:
- `source_exchange_id` and/or `source_pack_id` MUST be present and pinned
- tasks MUST NOT use floating pointers like “latest” for correctness-critical steps

### 7.2 “Human approval” rule
If `require_human_approval == true`:
- Orgo MUST prevent dispatch until approval is recorded
- SwarmCraft MUST not bypass approval gates

### 7.3 “Forbidden tools” rule
If a task lists forbidden tools:
- Orgo MUST prevent dispatch if method/tool violates it
- SwarmCraft MUST validate at runtime and fail deterministically if violated

### 7.4 “Artifact discipline” rule
If a task declares expected outputs:
- SwarmCraft MUST either produce them or fail with a structured error code
- Orgo MUST record produced artifacts and bind them to task lineage

---

## 8) Failure modes (expected behavior)

### 8.1 Gate failure (Orgo)
- Task remains `BLOCKED` (or `DRAFT`) with reason codes
- No dispatch occurs

### 8.2 Execution failure (SwarmCraft)
- Task transitions to `FAILED`
- SwarmCraft emits `TASK_FAILED` with:
  - deterministic `error_code`
  - step diagnostics
  - partial artifacts (if any) explicitly marked
- Orgo records failure, may open remediation tasks

### 8.3 Partial execution
Allowed only if policy permits:
- Task may complete with `DONE` but must record partial outputs and warnings
- Orgo must label the task outcome as `PARTIAL` (or equivalent tag)

### 8.4 Idempotency and retries
- If retries are allowed, SwarmCraft must:
  - use correlation IDs and attempt IDs
  - avoid duplicating side effects without explicit idempotency strategy
- Orgo must enforce `max_attempts`

---

## 9) Observability and audit requirements

### 9.1 Correlation IDs
- Every telemetry event MUST include:
  - `task_id`
  - `case_id`
  - `tenant_id`
  - `correlation_id`
  - `attempt_id`
  - timestamp
  - producer version (SwarmCraft engine version)

### 9.2 Artifact lineage
For each produced artifact:
- include `artifact_type`, `artifact_id`, `hash`
- include producing `task_id` and `attempt_id`
- include upstream context refs when relevant (plan_id, exchange_id, pack_id)

### 9.3 Minimum metrics
- task throughput by routing/priority
- dispatch latency
- execution duration distribution
- failure rate by error_code
- retry rate and success-after-retry
- artifact production counts by type

---

## 10) Security and access control

- Orgo is the policy authority for what is permitted to execute.
- SwarmCraft must run under a constrained identity and least-privilege permissions.
- Confidential artifacts must carry confidentiality tags; access is enforced by Orgo/Konnaxion policies.
- Telemetry must not leak protected data; use opaque artifact refs where necessary.

---

## 11) Conformance tests (required)

A conformant implementation MUST pass:

### 11.1 Orgo
- refuses to dispatch tasks not in `READY`
- blocks tasks with unpinned canon/pack refs
- enforces tool allow/deny lists
- enforces human approval gates
- records telemetry and binds artifacts to tasks deterministically

### 11.2 SwarmCraft
- emits required telemetry events with required fields
- respects task constraints (max_attempts, approvals, tool restrictions)
- produces declared outputs or fails with deterministic error codes
- maintains correlation/attempt identity across retries
- does not write to Kristal Exchange (no canon mutation path)

---

## 12) Minimal examples

### Example A: deterministic tool task
- Task: “Generate validation report for dataset X”
- Inputs: `resolved_claim_ir_id`
- Outputs: `validation_report`
- Method: tool
- Gate: pinned blueprint/policy versions
- Telemetry: TASK_STARTED → STEP_* → ARTIFACT_PRODUCED → TASK_FINISHED

### Example B: human approval
- Task requires `require_human_approval=true`
- Orgo blocks dispatch until approval artifact is attached
- SwarmCraft refuses execution without approval reference
