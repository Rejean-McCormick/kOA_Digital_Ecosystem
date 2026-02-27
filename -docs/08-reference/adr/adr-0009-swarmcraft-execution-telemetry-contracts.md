# ADR-0009: SwarmCraft Execution & Telemetry Contracts

**Status:** Accepted
**Date:** 2026-02-09
**Scope:** Orgo ⇄ SwarmCraft execution boundary; telemetry emitted upstream to Orgo and (optionally) Architect.

---

## 1. Context

SwarmCraft is the execution engine that turns an Architect plan into real outputs (files, deployments, transformations, human contributions). It is explicitly downstream of planning and upstream of deliverables, and it must report status/results back to Orgo/Architect. 

Because execution is the “activity channel,” the system requires: (a) faithful execution without drift, (b) a strict lifecycle for typed work objects (Cases/Tasks), and (c) end-to-end traceability through deterministic, structured telemetry. 

---

## 2. Decision

### 2.1 Canonical execution contract (Task Envelope)

SwarmCraft SHALL execute only **typed work objects** (Tasks) issued by Orgo (originating from Architect plans). Task definitions are **immutable** once dispatched: SwarmCraft does not silently reinterpret goals, reorder dependencies, or change parameters; if a change is needed, SwarmCraft requests a new/updated task upstream. 

SwarmCraft SHALL:

* **Validate + acknowledge** tasks before execution (inputs exist, agent available, schema/version compatible), otherwise fail fast with an explicit rejection. 
* Enforce **faithful execution / no unauthorized improvisation**: do what the task specifies or fail explicitly; no silent skipping. 
* Preserve **dependency order**; parallelize only where declared safe by the plan. 
* Enforce **atomicity**: each task ends in a terminal outcome (success/failure/cancel) with a defined result or explicit error. 
* Enforce **resource compliance** (timeouts/budgets/deadlines/rate-limits) when provided by Orgo. 

### 2.2 Canonical telemetry contract (Execution Events)

SwarmCraft SHALL emit structured telemetry for each task: start/end/outcome and key metadata, including correlation identifiers to link execution back to the originating request. 

Telemetry SHALL be compatible with the ecosystem’s structured logging and correlation-ID guidance:

* Propagate **build_id** and **tenant_id** across service boundaries / message metadata. 
* Emit stable structured log fields (ts, service, stage, event, build_id, tenant_id, attempt, duration, error_code, etc.). 
* Follow privacy guidance: avoid raw sensitive content in logs; prefer hashes/pointers; redact where needed. 

### 2.3 Orgo lifecycle alignment

SwarmCraft SHALL map execution state to Orgo’s task lifecycle model:

* Task statuses include `PENDING | IN_PROGRESS | ON_HOLD | COMPLETED | FAILED | ESCALATED | CANCELLED`. 
* Transitions must obey the allowed state machine; terminal states are terminal; `closed_at` is set upon terminal transitions. 
* Orgo’s append-only task event log is the system audit surface; SwarmCraft events must be ingestible as TaskEvents.

### 2.4 Non-goals

* Do **not** embed operational telemetry as first-class truth inside Kristal Exchange/Runtime Pack schemas; keep execution telemetry operational and link via IDs. 
* SwarmCraft does **not** mutate Kristal truth directly; if execution yields new knowledge, it must re-enter governance via Orgo → SenTient → Kristal. 

---

## 3. Specification

### 3.1 Task Envelope (minimum required fields)

**Canonical identifiers**

* `task_id` (UUID; primary execution correlation ID)
* `build_id` (UUID; ties to the end-to-end governed run) 
* `tenant_id` (if multi-tenant) 
* `case_id` (optional; Orgo case linkage)

**Execution definition**

* `agent_kind` (enum or string; e.g., `ai_service`, `human`, `tooling`)
* `agent_selector` (routing info: role, capability class, or concrete worker)
* `instructions` (task objective and acceptance criteria; must be unambiguous) 
* `inputs[]` (list of input refs; prefer content-hash pointers)
* `outputs[]` (declared output artifacts; paths/URIs + expected format)

**Constraints**

* `depends_on[]` (task_ids)
* `timeout_ms` / `budget` / `due_at` (optional)
* `visibility` / handling flags (optional; aligns with Orgo confidentiality patterns)

**Immutability rule**

* SwarmCraft MUST treat this envelope as immutable; if any core field must change, request a new task. 

### 3.2 Task outcomes (atomic results)

Each task ends in exactly one terminal outcome:

* `COMPLETED`: outputs produced and meet acceptance criteria
* `FAILED`: explicit error, no silent partial acceptance
* `CANCELLED`: explicitly terminated

Atomicity is required because downstream orchestration depends on binary per-task outcomes. 

### 3.3 Telemetry event schema (minimum)

All SwarmCraft events MUST be structured and must carry correlation IDs:

```json
{
  "ts": "RFC3339",
  "service": "swarmcraft",
  "event": "TASK_STARTED|TASK_COMPLETED|TASK_FAILED|TASK_RETRY|TASK_CANCELLED|TASK_ON_HOLD|TASK_ESCALATED",
  "stage": "execute",
  "task_id": "uuid",
  "build_id": "uuid",
  "tenant_id": "opaque-id",
  "attempt": 1,
  "agent_id": "optional",
  "duration_ms": 1234,
  "status": "IN_PROGRESS|COMPLETED|FAILED|...",
  "error_code": "optional",
  "error_message": "bounded, optional",
  "output_refs": ["sha256:...", "uri-hash:..."],
  "metrics": { "optional": "bounded" }
}
```

**Required emissions**

* `TASK_STARTED` when execution begins
* `TASK_COMPLETED` or `TASK_FAILED` exactly once per attempt
* If retrying, emit `TASK_RETRY` with incremented `attempt`

Telemetry must include task lifecycle events with timestamps, agent IDs, and error codes (if failed), and must be tagged by `task_id` plus upstream correlation (`build_id` / request id) for end-to-end tracing. 

### 3.4 Correlation and propagation rules

* Orgo generates `build_id` and propagates it downstream; SwarmCraft must preserve and echo it in every event/log. 
* Structured log field minimums should match ecosystem guidance (ts/level/service/stage/event/build_id/tenant_id/etc.). 

### 3.5 Privacy and safety rules (telemetry)

* Do not log raw extracted text by default; prefer hashed references/pointers.
* Avoid signatures/private key material in logs.
* Redact or hash tenant identifiers in public logs when needed. 

---

## 4. Implementation notes

### 4.1 AI-call normalization (optional but recommended within SwarmCraft)

When SwarmCraft dispatches tasks to AI services, normalize provider outputs into a stable response object (status/data/tool_calls/usage) to prevent API drift from breaking the orchestrator. 

### 4.2 Orgo database alignment

* SwarmCraft events should map cleanly into Orgo’s `task_events` append-only log and update `tasks.status` using the canonical state machine. 
* On terminal states, ensure `closed_at` is set at the transition timestamp. 

---

## 5. Consequences

**Positive**

* Enforces “execution without drift” and makes failures explicit and actionable upstream. 
* Enables deterministic debugging/audit through end-to-end correlation IDs and structured logs. 
* Makes parallel execution safe by requiring declared dependencies and consistent lifecycle handling. 

**Negative**

* Requires discipline: versioned envelope/event schemas, consistent propagation, and storage/retention for telemetry.
* Adds operational overhead (event ingestion, dashboards, privacy controls).

---

## 6. Open items

* Formalize a versioned **Task Envelope schema** (JSON Schema) and register it in the artifact catalog (this ADR defines the minimum required fields; the schema should become canonical).
* Define a bounded shared **error_code taxonomy** for SwarmCraft (aligned with ecosystem stage-level error codes).

---
