# Orgo Task

**Normative for kOA:** YES
**External normative reference:** None (may reference Kristal artifact IDs as opaque refs)

An **Orgo Task** is the **atomic, executable unit of governed work** in kOA. Nothing executes unless it is expressed as an Orgo Task in an executable state.

Schema: `30-artifacts/schemas/orgo-task.schema.json`

## Purpose

An Orgo Task provides:

* a single executable contract for SwarmCraft (or any executor)
* explicit inputs/expected outputs at the artifact-type level
* deterministic gating rules (what must be true before dispatch)
* auditability (who/what/why/when), including lineage to mandate/blueprint/canonical refs
* a status lifecycle with allowed transitions and structured reasons

## Core invariants

* **Governance before execution:** tasks must pass Orgo gates before dispatch.
* **Pinned context:** tasks must reference pinned mandate/blueprint; and when relevant pinned external artifact IDs (no “latest”).
* **Auditability:** execution must emit telemetry with correlation + attempt identity and bind produced artifacts to the task.
* **No canon mutation:** task outcomes may produce new artifacts and new work, but must not mutate external canon directly.

## Identity and scope

Minimum identity fields:

* `task_id` (globally unique, stable)
* `case_id` (required; tasks always belong to a Case)
* `tenant_id` (required)
* `organization_id` (required in multi-tenant org-scoped deployments)

Recommended:

* `title` (human-readable)
* `created_at`, `created_by`
* `labels[]` (routing and filtering)
* `confidentiality` (visibility rules)

## Status lifecycle

A Task has a single `status` and optional `status_reason` + `reason_codes[]`.

Recommended statuses:

* `DRAFT` — incomplete; not executable; editable
* `BLOCKED` — cannot become executable until gates are satisfied; must include reasons
* `READY` — executable; can be dispatched
* `RUNNING` — execution in progress; bound to an `attempt_id`
* `DONE` — terminal success
* `FAILED` — terminal failure (with deterministic `error_code`)
* `CANCELED` — terminal canceled
* `PAUSED` — optional; not dispatched until resumed

Allowed transition sketch (policy may further restrict):

* `DRAFT → READY | BLOCKED | CANCELED`
* `BLOCKED → DRAFT | READY | CANCELED`
* `READY → RUNNING | BLOCKED | CANCELED`
* `RUNNING → DONE | FAILED | PAUSED`
* `PAUSED → READY | CANCELED`

Orgo MUST NOT set a task directly to `RUNNING` unless it is dispatching it.

## Routing and priority

Routing is used for ownership, queues, and policy:

* `routing.base`
* `routing.category`
* `routing.subcategory`
* optional `routing.role` (horizontal axis)

Priority is discrete, e.g. `P0`–`P4`.

## Method (how the task is executed)

`method.type` is one of:

* `tool` — deterministic tool execution
* `agent` — agentic execution under explicit constraints
* `human` — requires human action
* `hybrid` — composed, must still be auditable

Method includes:

* `method.name` (executor/tool/agent identifier)
* `method.params` (structured parameters; must be stable for determinism)
* optional `method.capabilities[]` (required capabilities)

## Inputs and expected outputs

Inputs are artifact references, not embedded payloads:

* `inputs[]`: list of `{ artifact_type, artifact_id, hash? }`

Expected outputs are declarations of what must be produced:

* `expected_outputs[]`: list of `{ artifact_type, artifact_id? }`

If `expected_outputs[]` is present, the executor MUST either:

* produce the outputs and emit `ARTIFACT_PRODUCED`, or
* fail with a deterministic `error_code`.

## Constraints

Tasks may include constraints such as:

* `deadline_at`
* `max_attempts`
* `retry_policy` (deterministic)
* `require_human_approval` (boolean)
* `tool_allowlist[]` / `tool_denylist[]`
* `time_budget_ms` / `cpu_budget` (if applicable)
* `partial_ok` (only if policy allows partial completion)

## Gates (executable-state requirements)

A Task is executable (`READY`) only if all applicable gates pass:

1. **Schema valid** (task conforms to `orgo-task.schema.json`)
2. **Pinned references**

   * `mandate_id` and blueprint reference are pinned
   * any external/canonical/runtime refs are explicit IDs (no floating “latest”)
3. **Capability constraints**

   * method/tool/agent is allowed by mandate/blueprint/policy
4. **Approval gates**

   * if `require_human_approval=true`, Orgo must record approval before dispatch
5. **Attempts**

   * dispatch only if `attempt_count < max_attempts` (if set)

If any gate fails, Orgo keeps the task `DRAFT` or `BLOCKED` with structured reasons and MUST NOT dispatch.

## Telemetry contract (required)

Tasks define minimum telemetry expectations so Orgo can reconcile status deterministically.

Minimum required fields on every execution event:

* `task_id`, `case_id`, `tenant_id`
* `correlation_id` (stable per dispatch chain)
* `attempt_id` (unique per run)
* `timestamp`
* `producer` (executor identity/version)

Minimum event types:

* `TASK_STARTED`
* `STEP_STARTED` (optional but recommended for multi-step methods)
* `STEP_FINISHED`
* `ARTIFACT_PRODUCED` (for each produced artifact; includes type/id/hash)
* `TASK_FINISHED` or `TASK_FAILED`

## Results and reconciliation

Executors return (or Orgo derives from telemetry):

* terminal status (`DONE` / `FAILED` / `CANCELED`)
* `error_code` (stable, deterministic on failures)
* `diagnostics` (structured)
* `produced_artifacts[]` (type/id/hash)
* `warnings[]` (structured)

Orgo MUST bind produced artifacts to the Task record and update Case-level rollups.

## Relationships

Orgo Task commonly references:

* `mandate_id` (from Mandate Bundle)
* `blueprint_id` / `blueprint_version`
* upstream work: `plan_id` (if used)
* external artifact refs (opaque IDs)
* downstream artifacts: Build Record / Release Record IDs (kOA-owned)

## Minimal example

```json
{
  "schema_version": "1",
  "task_id": "task:01HTQ2W6JY7Y9KZC6B0M9Y2B3A",
  "case_id": "case:01HTQ2W4QZ0K4B2R3F2M6X9H1P",
  "tenant_id": "tenant:acme",
  "organization_id": "org:acme",
  "title": "Validate resolved claims for mandate v12",
  "status": "READY",
  "priority": "P2",
  "routing": { "base": "build", "category": "validate", "subcategory": "claims" },
  "method": {
    "type": "tool",
    "name": "validator",
    "params": { "profile": "default" }
  },
  "inputs": [
    { "artifact_type": "resolved-claims", "artifact_id": "rcir:01HTQ2..." }
  ],
  "expected_outputs": [
    { "artifact_type": "validation-report" }
  ],
  "constraints": {
    "max_attempts": 2,
    "require_human_approval": false,
    "tool_allowlist": ["validator"]
  },
  "lineage": {
    "mandate_id": "mandate:01HTQ1...",
    "blueprint_id": "blueprint:01HTQ1...:v12",
    "source_refs": [
      { "ref_type": "external", "ref_id": "exchange:01ABC..." }
    ]
  },
  "telemetry_contract": {
    "require_correlation_id": true,
    "required_events": [
      "TASK_STARTED",
      "STEP_STARTED",
      "STEP_FINISHED",
      "ARTIFACT_PRODUCED",
      "TASK_FINISHED",
      "TASK_FAILED"
    ]
  },
  "created_at": "2026-02-27T14:05:00Z",
  "created_by": "user:ops"
}
```
