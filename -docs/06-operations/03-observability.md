# Observability (Operations)

This document defines the observability contract for the ecosystem: what must be logged, measured, traced, and audited across the lifecycle so that builds, releases, renders, executions, and feedback cycles remain reproducible, diagnosable, and governance-compliant.

Observability is **non-mutating**: telemetry MUST NOT edit canon; it produces signals that may open governed work (Cases/Tasks).

---

## 1) Principles

1) **End-to-end correlation**: every event MUST be linkable across nodes and stages.
2) **Artifact lineage first**: artifact IDs are the primary join keys.
3) **Deterministic gates are explainable**: gate outcomes MUST include reason codes + minimal decision trace.
4) **Non-leaking**: telemetry MUST NOT leak protected data; prefer opaque refs.
5) **Stage completeness**: every stage MUST emit start/finish + outcome.
6) **Replayability**: logs MUST capture versions/config hashes required to reproduce outcomes.
7) **Stable envelopes**: telemetry envelopes SHOULD be schema-versioned; fields are additive-only.

---

## 2) Identity and correlation model (required)

### 2.1 Required identifiers (minimum)
Every telemetry event MUST include:

- `tenant_id`
- `environment` (e.g., `prod|staging|dev`)
- `correlation_id` (stable across a chain of events)
- `producer` (component name)
- `producer_version`
- `timestamp` (RFC3339)
- `severity` (`DEBUG|INFO|WARN|ERROR`)
- `event` (string enum)

And SHOULD include when applicable:

- `build_id` (pipeline build correlation)
- `stage_id` (`ingest|extract|resolve|validate|compile|publish|render|execute|feedback`)
- `attempt_id` (monotonic per stage/task attempt)
- `config_hash` (when output-affecting)
- `policy_id` and `blueprint_hash` (when gating/producing artifacts)

### 2.2 Canonical join keys (recommended)
Prefer joining by stable artifact/work identifiers:

- `exchange_id` (Kristal Exchange / `kristal_id`)
- `pack_id` (Runtime Pack)
- `render_bundle_id` (Render Bundle)
- `claim_ir_ref`, `resolved_claim_ir_ref`
- `validation_report_id` (or hash/ref)
- `case_id`, `task_id` (Orgo)
- `feedback_id`

---

## 3) Event taxonomy (standardized)

All components SHOULD emit events using a shared envelope (schema-versioned):

```json
{
  "schema_version": "1.0",

  "event": "STRING_ENUM",
  "timestamp": "2026-02-09T00:00:00Z",
  "severity": "INFO",

  "tenant_id": "t_...",
  "environment": "prod",

  "producer": "Orgo|Kristal|SenTient|Architect-Render|Architect-Strategy|SwarmCraft|Konnaxion|EkoH",
  "producer_version": "x.y.z",
  "config_hash": "sha256:...",

  "correlation_id": "c_...",
  "attempt_id": "a_...",

  "stage_id": "ingest|extract|resolve|validate|compile|publish|render|execute|feedback",
  "build_id": "b_...",

  "refs": {
    "exchange_id": "",
    "pack_id": "",
    "render_bundle_id": "",
    "claim_ir_ref": "",
    "resolved_claim_ir_ref": "",
    "validation_report_id": "",
    "case_id": "",
    "task_id": "",
    "feedback_id": ""
  },

  "policy": {
    "policy_id": "",
    "blueprint_hash": ""
  },

  "outcome": {
    "status": "SUCCESS|FAIL|BLOCKED|REFUSED|PARTIAL",
    "error_code": null,
    "reason_codes": [],
    "diagnostics_ref": ""
  },

  "metrics": {},
  "tags": {}
}
````

### 3.1 Build lifecycle events (Orgo + Kristal)

* `BUILD_STARTED`
* `STAGE_STARTED`
* `STAGE_FINISHED`
* `BUILD_FINISHED`
* `BUILD_FAILED`

### 3.2 Validation gate events (Orgo)

* `VALIDATION_STARTED`
* `VALIDATION_PASSED`
* `VALIDATION_PARTIAL`
* `VALIDATION_FAILED`

### 3.3 Compilation events (Kristal)

* `COMPILE_STARTED`
* `COMPILE_REFUSED` (e.g., missing validation PASS)
* `EXCHANGE_WRITTEN`
* `PACK_WRITTEN`
* `COMPILE_FINISHED`

### 3.4 Distribution events (Konnaxion)

* `PACK_PUBLISHED`
* `PACK_DOWNLOAD_STARTED`
* `PACK_DOWNLOAD_FINISHED`
* `PACK_VERIFY_PASSED`
* `PACK_VERIFY_FAILED`
* `PACK_ACTIVATION_SUCCEEDED`
* `PACK_ACTIVATION_FAILED`
* `PACK_ROLLBACK_TRIGGERED`
* `PACK_ROLLBACK_SUCCEEDED`
* `PACK_ROLLBACK_FAILED`

### 3.5 Rendering events (Architect-Render)

* `RENDER_STARTED`
* `RENDER_FINISHED`
* `RENDER_PARTIAL`
* `RENDER_REFUSED` (e.g., trace gap, unpinned inputs)
* `TRACE_COVERAGE_REPORTED` (assertions_total, traced, omitted, uncertain, refused)

### 3.6 Execution events (SwarmCraft)

Minimum aligns with Orgo task telemetry contract:

* `TASK_STARTED`
* `STEP_STARTED`
* `STEP_FINISHED`
* `ARTIFACT_PRODUCED`
* `TASK_FINISHED`
* `TASK_FAILED`

### 3.7 Feedback events (Konnaxion + Orgo)

* `FEEDBACK_RECEIVED`
* `FEEDBACK_TRIAGED`
* `FEEDBACK_REQUEST_INFO`
* `FEEDBACK_CASE_CREATED`
* `FEEDBACK_RESOLVED`

---

## 4) Component-specific required telemetry

### 4.1 Orgo (control plane)

Orgo MUST emit:

* build stage ordering events (`STAGE_*`)
* gate decision events with structured reason codes and policy/blueprint pins
* task lifecycle transitions and dispatch outcomes
* routing label at creation (no sensitive content)
* linkage maps (as refs in events or as separate linkage events):

  * `plan_id → case_id → task_id`
  * `task_id → produced artifacts`
  * `feedback_id → created work items`

Minimum required fields when gating:

* `policy.policy_id`, `policy.blueprint_hash`
* `outcome.reason_codes[]` (deterministic categories)
* `refs.validation_report_id` for validation outcomes

### 4.2 Kristal (canon pivot)

Kristal MUST emit:

* compile start/finish
* refusal events when validation is not PASS (with deterministic reason codes)
* resulting `exchange_id`, `pack_id`, and manifest refs/hashes
* size metrics (files_count, bytes_total) for Exchange and Pack
* optional reproducibility checks (sampled rebuild comparisons), reported as metrics/tags

### 4.3 SenTient (resolution)

SenTient SHOULD emit:

* counts of resolved/unresolved/rejected/merged
* conflict group stats and ambiguity group counts
* policy versions used for resolution (`policy_id`, `blueprint_hash`)
* deterministic mode flags and seed policy (if any)

### 4.4 Architect-Strategy

Architect-Strategy SHOULD emit:

* plan creation events, version/revision triggers
* task graph stats (counts by routing/type/category)
* refusal/blocked reasons (unpinned inputs, missing mandate, conflicts)

### 4.5 Architect-Render

Architect-Render MUST emit:

* deterministic context: template id/version, locale, strictness profile, config hash
* trace coverage stats:

  * `assertions_total`
  * `assertions_traced`
  * `assertions_omitted`
  * `assertions_uncertain`
  * `assertions_refused`
  * `ambiguities_rendered`
* refusal diagnostics:

  * `error_code` (e.g., `TRACE_GAP`, `UNPINNED_INPUT`, `POLICY_BLOCK`)
  * minimal offending span/anchor references (no protected excerpts)

### 4.6 SwarmCraft (execution)

SwarmCraft MUST emit:

* task lifecycle events listed in 3.6
* step-level timing and error codes
* artifact production events with:

  * `artifact_type`, `artifact_id`/ref, `hash`
  * `task_id`, `attempt_id`
* runtime identity:

  * engine version, tool versions (if applicable)
  * environment tag (`prod|staging`)

### 4.7 Konnaxion (distribution + edge)

Konnaxion MUST emit:

* verification outcomes (pass/fail) and deterministic cause codes
* activation/rollback outcomes
* cache/eviction events (without exposing user content)
* optional offline/stale indicators and serving mode tags

### 4.8 EkoH (ledger)

EkoH SHOULD emit:

* score update events (append-only history pointers)
* vote aggregation completion events (counts, weighted sums)
* confidentiality mode transitions (no raw private values)

---

## 5) Metrics (minimum set)

### 5.1 Pipeline metrics

* build throughput (builds/day) by tenant/channel
* stage durations (p50/p95/p99) per stage
* validation pass/partial/fail rates; top failure reason codes
* compile success rate; compile refusal rate
* artifact sizes (Exchange/Pack) trends

### 5.2 Rendering metrics

* render success/partial/refusal rates by strictness profile
* trace coverage distribution (traced/total; refused/omitted/uncertain shares)
* top trace-gap reasons
* output sizes and render times

### 5.3 Distribution metrics

* download success rate
* verification failure rate (by cause)
* activation failure rate
* rollback rate and rollback success rate
* cache hit ratio; storage usage; eviction counts

### 5.4 Execution metrics

* task dispatch latency (Orgo → SwarmCraft)
* task durations by routing and priority
* failure rate by `error_code`
* retry rate and success-after-retry
* artifact production counts by type

### 5.5 Feedback metrics

* feedback volume by type (truth/policy/ux/ops)
* time-to-triage, time-to-resolution
* conversion rate: feedback → new canon (new `exchange_id`)
* duplicate rate; spam quarantine rate (if applicable)

---

## 6) Tracing (recommended)

### 6.1 Trace strategy

Implement distributed traces where the root span is:

* `build_id` for pipeline operations, or
* `task_id` for execution operations, or
* `render_bundle_id` for render operations.

### 6.2 Required trace attributes

Attach:

* `tenant_id`, `environment`, `component`, `version`
* artifact IDs (`exchange_id`, `pack_id`, etc.)
* policy pins (`policy_id`, `blueprint_hash`)
* gate decision codes (as span events)

---

## 7) Dashboards (recommended)

1. **Pipeline health**: stage durations, validation failures, compile refusals
2. **Truth boundary**: validation pass/partial/fail breakdown, top reason codes
3. **Render integrity**: refusal rate, trace coverage distribution
4. **Distribution safety**: verification failures, activation failures, rollback events
5. **Execution reliability**: task SLA by priority, failure clusters, retries
6. **Feedback operations**: triage/resolution queues, truth-impacting throughput

---

## 8) Alerts (minimum)

### 8.1 Severity P0 (page)

* validation suddenly fails > X% across tenants (regression)
* compile occurs when validation is not PASS (should be impossible)
* pack verification failure spikes (possible key/index compromise)
* rollback failure rate spikes (cannot recover)
* trace-gap refusals spike (render integrity broken)
* task dispatch latency exceeds threshold for P0/P1

### 8.2 Severity P1 (ticket)

* gradual degradation of stage durations
* rising cache eviction pressure
* increasing duplicate feedback without resolution capacity

All alerts MUST include:

* top reason codes
* correlated artifact/work IDs
* links to traces/logs via `correlation_id` (and `build_id`/`task_id` where applicable)

---

## 9) Retention and privacy

### 9.1 Retention (policy-defined)

Minimum recommended retention windows:

* audit logs (policy/gate outcomes): long retention
* operational logs: medium retention
* high-volume telemetry (per-step): shorter retention with aggregation

### 9.2 Privacy constraints

* Do not log raw user content by default.
* Prefer artifact refs and opaque provenance pointers.
* Apply confidentiality tags to telemetry and enforce access controls.
* If public vs private traces are required, split telemetry/traces by visibility policy.

---

## 10) Conformance tests (required)

A conformant deployment MUST prove:

* every stage emits start/finish with outcome
* every gate emits reason codes and policy pins
* artifact production events include IDs/refs and hashes when applicable
* `correlation_id` links at least: Orgo ↔ Kristal ↔ Konnaxion ↔ Architect ↔ SwarmCraft
* telemetry never writes to Kristal Exchange
* redaction rules prevent leaking protected data in logs/traces

