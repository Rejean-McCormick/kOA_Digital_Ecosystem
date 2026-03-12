# Build Record (kOA)

**Artifact class:** Operational (governance + reproducibility)  
**Owned by:** Orgo (control plane)  
**Schema:** `docs/30-artifacts/schemas/build-record.schema.json`  
**Purpose:** Provide a single, auditable, content-addressed (or at least tamper-evident) record of *what was built*, *from what*, *under which pinned policies/blueprints*, *which gates ran*, and *what artifacts were produced*.

This record is kOA-owned. It references Kristal artifacts by **opaque IDs + manifest refs**, but does not redefine Kristal contracts.

---

## 1) Producer → Consumers

**Producer:** Orgo (pipeline orchestrator)  
**Consumers:** Audit tooling, release tooling, CI/CD, operators, conformance tests, incident response

---

## 2) Identity

- `build_id` is the primary identifier (kOA namespace).
- Orgo SHOULD additionally compute a `record_hash` over a canonicalized representation of the Build Record (kOA-defined) and optionally sign it.

---

## 3) What the Build Record must answer

A Build Record MUST allow an auditor to reconstruct:

1) **Inputs**: exactly which upstream input snapshots were used (content-addressed refs).  
2) **Pinned environment**: blueprint/mandate/policy selections and toolchain identity.  
3) **Stage spine**: which stages ran, in what order, with what outcomes.  
4) **Gate decisions**: validation results and any hard-stop reasons.  
5) **Outputs**: the produced Kristal artifacts (Exchange + Runtime Pack) and their manifest references.

---

## 4) Minimum required sections (contract)

### 4.1 Header
- `build_id`
- `created_at`
- `actor` (who/what triggered it; human, scheduler, CI)
- `pipeline` (pipeline name + version)
- `environment` (workspace/cluster, optional but recommended)

### 4.2 Pinned context
- `mandate_ref` (kOA mandate bundle ref; may be absent for dev)
- `blueprint_ref` (pinned blueprint bundle ref)
- `policy_selections` (kOA list of enabled policies/profiles)

### 4.3 Inputs
- `input_snapshots[]` (content-addressed refs)
- Optional: `input_summary` (counts, high-level metadata, no raw content)

### 4.4 Stages (timeline)
A list of stage executions, each with:
- `stage` (enum/name)
- `started_at`, `ended_at`
- `status` (`PASS|FAIL|SKIP`)
- `artifacts_out[]` (refs produced by that stage)
- `metrics` (optional)
- `errors[]` (stable codes/messages)

### 4.5 Gate outcomes
- `validation_gate` result (pass/fail + reference to the Kristal Validation Report artifact)
- Any additional gates (policy gate, safety gate, compatibility gate), if used

### 4.6 Outputs (references only)
- `validation_report_ref` (Kristal artifact ref)
- `exchange_manifest_ref` (Kristal artifact ref)
- `exchange_id` (Kristal identifier; opaque string)
- `runtime_pack_manifest_ref` (Kristal artifact ref)
- `runtime_pack_id` (Kristal identifier; opaque string)

### 4.7 Integrity (recommended)
- `record_hash` (kOA hash object)
- `signatures[]` (kOA signature envelope; trust roots are kOA deployment policy)

---

## 5) Invariants

- **No compile on fail:** If validation fails, the Build Record MUST NOT contain Exchange/Runtime Pack output refs for that build attempt (or must mark them as absent and record a terminal failure).  
- **Reproducibility:** If the same `input_snapshots`, `blueprint_ref`, `policy_selections`, and toolchain versions are used, reruns MUST be comparable and explain any non-determinism via explicit recorded variance.  
- **Tamper evidence:** If `record_hash` is present, it MUST verify; if signatures are present, verification MUST be fail-closed for any workflow that treats the Build Record as authoritative (release, promotion, audit export).

---

## 6) Failure modes

- Missing or non-content-addressed `input_snapshots`
- Missing pinned `blueprint_ref` (for non-dev pipelines)
- Stage list does not contain required gates
- Output refs present despite failed validation gate
- Integrity block present but unverifiable

---

## 7) Minimal example (illustrative)

```json
{
  "build_id": "bld_2026_02_27_0007",
  "created_at": "2026-02-27T14:12:03Z",
  "actor": { "type": "ci", "id": "gha:org/repo#8421" },
  "pipeline": { "name": "orgo-pipeline", "version": "1.4.0" },

  "mandate_ref": "sha256:...mandate-bundle...",
  "blueprint_ref": "sha256:...blueprint-bundle...",
  "policy_selections": ["policy:baseline", "profile:strict-determinism"],

  "input_snapshots": ["sha256:...snap1...", "sha256:...snap2..."],

  "stages": [
    {
      "stage": "extract",
      "started_at": "2026-02-27T14:12:05Z",
      "ended_at": "2026-02-27T14:12:44Z",
      "status": "PASS",
      "artifacts_out": ["sha256:...claim-ir..."]
    },
    {
      "stage": "resolve",
      "started_at": "2026-02-27T14:12:45Z",
      "ended_at": "2026-02-27T14:13:21Z",
      "status": "PASS",
      "artifacts_out": ["sha256:...resolved-claim-ir..."]
    },
    {
      "stage": "validate",
      "started_at": "2026-02-27T14:13:22Z",
      "ended_at": "2026-02-27T14:13:58Z",
      "status": "PASS",
      "artifacts_out": ["sha256:...validation-report..."]
    },
    {
      "stage": "compile",
      "started_at": "2026-02-27T14:14:01Z",
      "ended_at": "2026-02-27T14:14:33Z",
      "status": "PASS",
      "artifacts_out": ["sha256:...exchange-manifest...", "sha256:...runtime-pack-manifest..."]
    }
  ],

  "validation_report_ref": "sha256:...validation-report...",
  "exchange_manifest_ref": "sha256:...exchange-manifest...",
  "exchange_id": "kristal:exchange:...",
  "runtime_pack_manifest_ref": "sha256:...runtime-pack-manifest...",
  "runtime_pack_id": "kristal:pack:...",

  "record_hash": { "alg": "sha256", "value": "..." }
}
````

---

## 8) Versioning and compatibility

* The Build Record schema is kOA-owned and versioned by kOA.
* Fields referencing Kristal artifacts (`exchange_*`, `runtime_pack_*`, `validation_report_ref`) are **opaque** and must not assume Kristal internal structure beyond “identifier + manifest ref”.

---


