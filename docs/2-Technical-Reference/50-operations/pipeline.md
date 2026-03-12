# Pipeline Operations (Orgo)

**Scope:** Operational runbook for the kOA pipeline as orchestrated by **Orgo**.  
**Non-goal:** This document does not restate Kristal artifact schemas. When a stage produces/consumes Kristal artifacts, it references them as opaque refs and points to `docs/40-integration/kristal-v4/contract-pointers.md`.

---

## 1) Stage spine (normative for kOA)

Orgo enforces this ordered stage spine:

1. **Ingest** → record `input_snapshots[]` (content-addressed refs)
2. **Extract** → produce Claim-IR (Kristal artifact)
3. **Resolve** → produce Resolved Claim-IR (Kristal artifact)
4. **Validate** → produce Validation Report (Kristal artifact)
5. **Compile** → produce Exchange + Runtime Pack (Kristal artifacts)
6. **Distribute** → publish pack(s) to channels (kOA)
7. **Activate** → verify + switch active pack (Konnaxion)
8. **Observe** → metrics/events/logs (kOA)
9. **Feedback** → cases/tasks (kOA)

**Hard rule:** If the Validate stage fails, Orgo MUST stop and MUST NOT run Compile (“no compile on fail”).

---

## 2) Required operational records (kOA)

For every pipeline execution, Orgo MUST persist a **Build Record**:

- `docs/30-artifacts/build-record.md`

For every promotion/publish action, Orgo MUST persist a **Release Record**:

- `docs/30-artifacts/release-record.md`

---

## 3) Inputs (Ingest)

### 3.1 What to verify at ingest
- Inputs are stored immutably and referenced by content hash
- Provenance recorded (source, timestamps, access controls)
- Any confidentiality policy applied before downstream processing

### 3.2 Failure handling
- If an input cannot be content-addressed or provenance is incomplete: fail the build attempt
- If access policy forbids use: exclude and record exclusion

---

## 4) Extract (Claim-IR)

### 4.1 Operational checks
- Extractor version is pinned and recorded
- Output is schema-valid per Kristal v4 (see pointers doc)
- Output is content-addressed and recorded

### 4.2 Failure handling
- If schema validation fails: stop build, record failure, open Orgo Case

---

## 5) Resolve (Resolved Claim-IR)

### 5.1 Operational checks
- Resolver version pinned and recorded
- Output is schema-valid per Kristal v4
- Ambiguity preserved; no silent coercion

### 5.2 Failure handling
- Stop build on schema invalid output
- Record deterministic error codes for repeated failures

---

## 6) Validate (Validation Report) — the acceptance gate

### 6.1 Operational checks
- Validator version pinned and recorded
- Report is schema-valid per Kristal v4
- Gate decision recorded in Build Record

### 6.2 Failure handling (hard)
- On validation failure:
  - Do **not** compile
  - Persist the Validation Report ref
  - Create an Orgo Case (triage) and attach the report

---

## 7) Compile (Exchange + Runtime Pack)

### 7.1 Preconditions
- Validation gate PASS
- Pinned blueprint + mandate + policy selections recorded
- Compiler version pinned and recorded

### 7.2 Operational checks
- Produced artifacts are schema-valid per Kristal v4
- Manifest refs recorded
- IDs/hashes/signatures verified at rest (spot-check or full verify per policy)

### 7.3 Failure handling
- If compile fails: record failure, do not distribute, open Orgo Case

---

## 8) Distribute (channels)

### 8.1 Publish rules
- Distribution is keyed by Release Record intent
- Packs are immutable; publishing creates new version identifiers

### 8.2 Failure handling
- If publish fails mid-way: mark release as failed and do not advance cohorts

---

## 9) Activate (Konnaxion)

### 9.1 Preconditions
- Pack fetched successfully
- Verification is **fail-closed**:
  - signature verification
  - hash verification
  - compatibility policy check

### 9.2 Activation rules
- Atomic switch of active pack
- Deterministic rollback to last-known-good on failure
- Downgrade prevention per compatibility policy

---

## 10) Observability (minimum signals)

### 10.1 Pipeline metrics
- Stage durations
- PASS/FAIL/SKIP counts
- Validation failure codes
- Compile failure codes

### 10.2 Distribution metrics
- Fetch success rate
- Verify failures (by reason)
- Activation/rollback events

### 10.3 Audit linkage
Every metric/event MUST reference:
- `build_id` and/or `release_id`
- associated artifact refs (where applicable)

---

## 11) Incident playbook triggers (examples)

- Repeated validation failures for the same blueprint/policy set
- Increase in activation verify failures
- Increase in rollback events
- Pack distribution channel drift (unexpected “latest”)

---

## 12) Operator checklist (quick)

- Identify `build_id` / `release_id`
- Retrieve Build/Release Records
- Retrieve referenced Kristal artifacts (Validation Report, manifests)
- Confirm gate outcomes and failure reasons
- If activation incident: confirm rollback executed and downgrade prevention state
- Create/attach Orgo Case and Tasks for remediation