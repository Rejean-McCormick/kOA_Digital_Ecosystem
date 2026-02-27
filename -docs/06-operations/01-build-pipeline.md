# Build Pipeline (Orgo Control Plane)

## Status
Normative (operational specification).

## Purpose
Define the canonical **Kristal build workflow** as executed and governed by **Orgo**:
- stage ordering
- deterministic gates (“no compile on fail”)
- build record requirements (audit + reproducibility)
- release/distribution tracking
- operator-facing behaviors (approvals, failures, replay)

This document specifies **how builds run** (contracts + gates + records), not the internal implementation of any component.

---

## 1) Scope and terminology

### Build
A **Build** is a single execution of the Kristal production pipeline that yields:
- **Kristal Exchange** (canonical truth artifact)
- **Runtime Pack** (portable offline derivative)
and optionally triggers **Release/Distribution**.

### Workflow
A **Workflow** is the Orgo-managed state machine that advances a Build through stages.

### Stage
A **Stage** is a named unit of work with:
- explicit input references (pinned)
- explicit output references (content-addressed when applicable)
- deterministic success/failure under pinned policy/config
- recorded timestamps, attempts, and operator actions (if any)

---

## 2) Mandatory stage order

A Kristal build workflow MUST follow this stage order:

1. **Ingest**
2. **Extract → Claim-IR**
3. **Resolve (SenTient) → Resolved Claim-IR**
4. **Validate → Validation Report**
5. **Compile → Exchange**
6. **Compile → Runtime Pack**
7. **Publish/Distribute**
8. **Post-publish verification** (RECOMMENDED)

Orgo MAY split stages into sub-stages, but MUST preserve:
- ordering
- gating semantics
- recorded artifact references per stage boundary

---

## 3) Stage definitions (inputs, outputs, and responsibilities)

### 3.1 Ingest
**Goal:** freeze input sources into immutable snapshots.

**Inputs**
- source connectors (docs, datasets, feeds, uploads)
- mandate context (required if the tenant/policy demands it)
- blueprint bundle reference (required if used downstream)

**Outputs**
- `input_snapshot_set_ref` (recommended: a snapshot-manifest ref that lists all snapshots)
- `input_snapshots[]` (content-addressed references)
- `ingest_report_ref` (optional)

**Rules**
- Snapshots MUST be immutable and referenceable.
- Provenance MUST be recorded per snapshot.
- Orgo MUST record the exact snapshot set used by the build (for replay).

---

### 3.2 Extract → Claim-IR
**Goal:** produce Claim-IR only (proposals with evidence pointers).

**Inputs**
- `input_snapshot_set_ref` or `input_snapshots[]`
- Claim-IR schema reference (pinned blueprint hash)
- extractor config/recipe ref (pinned)

**Outputs**
- `claim_ir_ref` (content-addressed)
- `extract_report_ref` (optional metrics/warnings)

**Rules**
- Extractors MUST output Claim-IR only (no Exchange writes, no canon mutation).
- Orgo MUST store Claim-IR references for audit and replay.

---

### 3.3 Resolve (SenTient) → Resolved Claim-IR
**Goal:** reconcile and normalize claims into explicit resolution states.

**Inputs**
- `claim_ir_ref`
- blueprint (ontology, normalization rules; pinned)
- resolution policy selections (pinned)
- optional hint/constraint payload refs (pinned if used)

**Outputs**
- `resolved_claim_ir_ref` (content-addressed)
- `resolution_report_ref` (optional)

**Rules**
- Resolution MUST preserve ambiguity explicitly (no forced disambiguation).
- Resolution MUST NOT write to Kristal directly.
- Orgo MUST store Resolved Claim-IR references for audit and replay.

---

### 3.4 Validate → Validation Report
**Goal:** deterministic acceptance gate for compilation.

**Inputs**
- `resolved_claim_ir_ref`
- blueprint validation rules + policy selections (pinned)
- validator identity/version/config hash (declared)

**Outputs**
- `validation_report_ref` (content-addressed)
- `validation_status` (passed|partial|failed) (redundant; derived from report)
- optional `validation_summary_ref`

**Rules**
- Validation MUST be deterministic under pinned policy/config.
- Validation MUST emit a report for every attempt (pass/fail/partial).
- Validation MUST use fail-closed behavior when required checks cannot be performed deterministically.

---

### 3.5 Compile → Exchange
**Goal:** compile canonical truth into Kristal Exchange.

**Inputs**
- `resolved_claim_ir_ref`
- `validation_report_ref` (MUST be PASS, unless policy explicitly allows PARTIAL)
- blueprint hash + policy_id (pinned)
- canonicalization profile reference (pinned)

**Outputs**
- `exchange_id` / `kristal_id`
- `exchange_manifest_ref` (content-addressed)
- `exchange_hash` + optional signatures

**Rules**
- Exchange artifacts MUST be immutable and content-addressed.
- Compilation MUST stamp policy/blueprint/canonicalization identity into the manifest.
- Compilation MUST refuse deterministically if validation is not allowed-by-policy.

---

### 3.6 Compile → Runtime Pack
**Goal:** derive an offline-executable pack from the Exchange.

**Inputs**
- Exchange (`exchange_id` + `exchange_manifest_ref`)
- pack profile selections (pinned)
- canonicalization profile reference (pinned)

**Outputs**
- `pack_id` / `runtime_pack_id`
- `runtime_pack_manifest_ref`
- payload hashes + optional signatures

**Rules**
- Runtime Pack MUST be verifiable (hash/signature refs).
- Pack policy selections MUST be recorded.
- Pack MUST be provably derived from a specific Exchange (manifest linkage).

---

### 3.7 Publish/Distribute
**Goal:** trigger distribution and track delivery/activation status.

**Inputs**
- `runtime_pack_manifest_ref` + payload refs
- distribution parameters (channels/cohorts/rollout policy; pinned)
- tenant signing/verification policy (pinned trust roots)

**Outputs**
- `release_id` (versioned release record)
- distribution status timeline (queued → delivered → verified → activated)
- deterministic failure codes (if any)

**Rules**
- Orgo MUST track distribution status and surface failures.
- Distribution systems MUST verify before activation (fail-closed).
- Orgo MUST treat verification failure as release failure (or policy-defined blocked state).

---

### 3.8 Post-publish verification (recommended)
**Goal:** verify published artifacts are retrievable, verifiable, and executable offline.

**Inputs**
- `release_id`
- channel/cohort state

**Outputs**
- `post_publish_report_ref`
- verification summary (pass/fail + codes)

**Rules**
- Verification SHOULD be deterministic.
- Verification SHOULD include integrity checks and offline execution checks.

---

## 4) Gating and determinism (mandatory)

### 4.1 Hard gate: “No compile on fail”
If validation fails:
- Orgo MUST mark the workflow as failed at **Validate**
- Orgo MUST NOT invoke **Compile → Exchange** or **Compile → Runtime Pack**
- Orgo MUST surface the validation report (codes + messages) to operators

If validation is `partial`:
- Orgo MUST consult policy; if partial is disallowed, treat as blocked/failed and do not compile.

### 4.2 Deterministic stage outputs (recorded refs)
For every stage, Orgo MUST record output references:
- `claim_ir_ref`
- `resolved_claim_ir_ref`
- `validation_report_ref`
- Exchange `exchange_id`/`kristal_id` + `exchange_manifest_ref`
- Runtime Pack `pack_id` + `runtime_pack_manifest_ref`

### 4.3 Idempotency and replay
Orgo SHOULD support deterministic replay:
- Given identical pinned snapshot sets and pinned config/policies, replay SHOULD yield identical outputs.
- If any upstream component is probabilistic, replay MUST use frozen intermediate snapshots (e.g., pinned Claim-IR), or must run that component under a declared deterministic mode.

Replay MUST NOT bypass integrity checks or gates.

---

## 5) Build Record (mandatory)

For every build, Orgo MUST persist a **Build Record** sufficient for audit and reproducibility.

### 5.1 Required fields

#### Identifiers
- `build_id` (unique)
- `tenant_id`
- `workflow_id`
- optional links:
  - `case_id` / `task_id` (if build is governed work)
  - `release_id` (if published)

#### Inputs (snapshots + config pins)
- `input_snapshot_set_ref` (recommended) and/or `input_snapshots[]`
- `config_refs[]` including:
  - blueprint bundle/hash
  - policy_id + policy selections ref
  - canonicalization profile ref
  - extractor/resolve/validate/compile recipes (if applicable)
- optional:
  - `claim_ir_ref` (if extractor output is externally produced/frozen)

#### Component identity + determinism surface
- for each component stage:
  - `component.name`, `component.version`, `config_hash`
  - determinism mode / seed policy (if applicable)

#### Outputs
- Exchange:
  - `exchange_id`/`kristal_id`
  - `exchange_manifest_ref`
  - `exchange_hash`
  - optional `signatures[]`
- Runtime Pack:
  - `pack_id`
  - `runtime_pack_manifest_ref`
  - payload hashes
  - optional `signatures[]`
- Validation:
  - `validation_report_ref`
  - `validation_status`

#### Stage timeline and audit
- per-stage:
  - `stage_name`
  - `status` (PENDING|RUNNING|SUCCEEDED|FAILED|BLOCKED)
  - `attempts[]` (attempt_id, started_at, ended_at, outputs, errors)
- `approvals[]` (if required)
- `operator_actions[]` (manual overrides, retries, aborts)

### 5.2 Minimal JSON shape (illustrative)
```json
{
  "build_id": "build-2026-02-09T152000Z-0001",
  "tenant_id": "tenant_demo",
  "workflow_id": "wf-01HZZ...",

  "case_id": "uuid-v4",
  "task_id": "uuid-v4",
  "release_id": "release_...",

  "inputs": {
    "input_snapshot_set_ref": "sha256:...",
    "input_snapshots": [{ "ref": "sha256:...", "kind": "dataset" }],
    "config_refs": [
      { "ref": "sha256:...", "kind": "blueprint_bundle" },
      { "ref": "sha256:...", "kind": "policy_selections" },
      { "ref": "sha256:...", "kind": "canonicalization_profile" }
    ]
  },

  "stages": [
    {
      "name": "INGEST",
      "status": "SUCCEEDED",
      "attempts": [
        {
          "attempt_id": "a1",
          "started_at": "2026-02-09T15:20:00Z",
          "ended_at": "2026-02-09T15:20:10Z",
          "outputs": ["sha256:snapshot_set"],
          "errors": []
        }
      ]
    },
    {
      "name": "VALIDATE",
      "status": "SUCCEEDED",
      "attempts": [
        {
          "attempt_id": "a1",
          "outputs": ["sha256:validation_report"],
          "errors": []
        }
      ]
    },
    {
      "name": "COMPILE_EXCHANGE",
      "status": "SUCCEEDED",
      "attempts": [
        {
          "attempt_id": "a1",
          "outputs": ["kristal:exchange:sha256:..."],
          "errors": []
        }
      ]
    }
  ],

  "outputs": {
    "exchange": {
      "exchange_id": "kristal:exchange:sha256:...",
      "exchange_hash": "sha256:...",
      "exchange_manifest_ref": "sha256:..."
    },
    "runtime_pack": {
      "pack_id": "kristal:pack:sha256:...",
      "runtime_pack_manifest_ref": "sha256:..."
    }
  }
}
````

---

## 6) Retries, failure handling, and operator workflow

### 6.1 Retry policy (recommended)

* Orgo SHOULD support bounded retries per stage.
* Retries MUST:

  * be recorded in the Build Record (attempt history)
  * preserve pinned input refs (or explicitly record changes)
  * never bypass validation gates

### 6.2 Failure semantics

* A failed stage MUST:

  * stop downstream stages
  * record deterministic error codes/messages
  * expose failing artifact refs (e.g., validation report)

### 6.3 Approvals (optional, governed)

If an approval is required:

* Orgo MUST pause at the stage boundary (typically before Publish/Distribute).
* Approval events MUST be recorded with actor identity and timestamp.

---

## 7) Multi-tenancy and security (mandatory)

* All build state MUST be scoped by `tenant_id` in multi-tenant deployments.
* Artifact references MUST be integrity verifiable (hash/signature references).
* Keys used for signatures SHOULD be tenant-scoped.
* Orgo MUST NOT mutate Exchange content outside the formal compile stage.
* Orgo MUST NOT allow any component to skip stages without Orgo advancing the workflow.

---

## 8) Observability (recommended)

Orgo SHOULD emit:

* counters per stage: started/succeeded/failed/blocked
* latency histograms per stage
* build success rate by tenant/policy/profile
* structured logs keyed by:

  * `build_id`, `workflow_id`, `tenant_id`
  * `exchange_id`, `pack_id`, `release_id`
* deterministic error dashboards driven by validation reason codes

---

## 9) Conformance tests (minimum)

An Orgo implementation claiming conformance MUST provide tests for:

* enforcing the mandatory stage order
* enforcing “no compile on fail”
* recording content-addressed refs for all stage outputs
* producing reproducible Build Records (inputs/config/policies pinned)
* replay does not bypass integrity checks
* release tracking records distribution outcomes and verification failures
* audit log contains stage transitions, retries, and approvals


