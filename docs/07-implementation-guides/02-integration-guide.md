# Integration Guide (Implementation)

This guide explains how to integrate a new module, tool, data source, projection, or node into the ecosystem **without breaking invariants**. The integration contract is always expressed as: **typed artifacts + deterministic gates + observability**.

---

## 1) Integration entry points (choose one)

### A) New producer (writes artifacts)
Examples:
- new extractor producing Claim-IR
- new resolver producing Resolved Claim-IR
- new compiler projection producing Runtime Pack files
- new renderer producing Render Bundles
- new execution worker producing telemetry + artifacts

### B) New consumer (reads artifacts)
Examples:
- a UI client consuming Runtime Packs and Render Bundles
- an internal service consuming Kristal Exchange
- an analytics service reading Orgo/telemetry (non-mutating)

### C) New workflow capability
Examples:
- new Orgo task method (agent/tool/human)
- new routing taxonomy / governance policy
- new distribution channel class (Konnaxion)

---

## 2) Non-negotiable integration rules (must hold)

1) **No canon mutation**: your integration must not write to Kristal Exchange.
2) **Truth boundary**: canon is produced only by Kristal after Orgo validation PASS.
3) **No new facts downstream**: if you render, you must trace; if you execute, you must not invent truth.
4) **Fail-closed on integrity**: distribution/activation must reject unverified or incompatible artifacts.
5) **Pinned inputs**: correctness-critical work must not depend on floating “latest.”
6) **Observability required**: all actions must emit correlation IDs, versions, and artifact lineage.

If any of these cannot be satisfied, the integration is not conformant.

---

## 3) Required artifacts and schemas (what you must implement)

### 3.1 If you WRITE an artifact
You MUST provide:
- a schema (JSON Schema or equivalent)
- a minimal example payload
- deterministic ordering/serialization rules (if applicable)
- a conformance test list
- an error code registry (deterministic)

Place in:
- `docs/04-artifacts-contracts/` (spec)
- `docs/04-artifacts-contracts/schemas/` (schema files)

### 3.2 If you READ an artifact
You MUST document:
- which artifact types you consume
- which fields are required vs optional
- how you handle missing fields / unknown versions
- what you do on verification failure (must be fail-closed where relevant)

Place in:
- your node doc under `docs/03-nodes/`
- and/or the relevant protocol doc under `docs/05-protocols-flows/`

---

## 4) Determinism requirements (by integration type)

### 4.1 Deterministic REQUIRED
- validation gates (Orgo)
- compilation outputs (Kristal Exchange + Runtime Pack manifests)
- rendering outputs in deterministic mode (Architect-Render)
- verification + activation (Konnaxion)

### 4.2 Deterministic OPTIONAL but auditable
- planning (Architect-Strategy) unless deterministic mode is enabled
- execution strategies that involve external systems (must be auditable + idempotent)

Minimum for non-deterministic components:
- record inputs, versions, config hash, and any seeds
- emit reason codes for decisions
- never use non-determinism to cross truth boundaries

---

## 5) Versioning and compatibility (must define)

Every integration MUST state its compatibility contract:
- `contract_version` it implements (consumer and producer)
- supported schema versions (min/max)
- migration behavior (upgrade/downgrade)
- compatibility failure behavior (must be deterministic)

If you ship Runtime Pack projections:
- declare `query_contract_version`
- declare required capabilities
- declare projection names and kinds

---

## 6) Security and privacy (must define)

### 6.1 Confidentiality
- classify artifacts (`public|internal|restricted|private`) if applicable
- ensure logs do not leak protected data (use opaque refs)

### 6.2 Integrity
If you distribute artifacts:
- implement hash/signature verification
- pin trust roots per channel
- fail closed on verification failure

### 6.3 Least privilege
Execution components must run with least privilege:
- tools allowlists/denylists
- restricted environments for external calls
- explicit human approval gates when required

---

## 7) Observability contract (mandatory)

Every integration MUST emit telemetry that includes:
- `tenant_id`
- `correlation_id`
- `attempt_id`
- component name + version
- `config_hash`
- `stage_id` (when part of pipeline)
- artifact refs (inputs/outputs) with IDs and hashes where applicable
- deterministic `error_code` and `reason_codes[]` on failure/refusal

Add required events to:
- `docs/06-operations/03-observability.md`

---

## 8) Integration process (step-by-step)

### Step 1 — Declare scope
- What node does this belong to?
- Does it produce artifacts, consume artifacts, or both?
- Which protocols does it participate in?

### Step 2 — Define contracts
- Select/define artifact schema(s)
- Define compatibility and versioning
- Define deterministic behavior (or auditable non-determinism boundaries)
- Define refusal/failure error codes

### Step 3 — Implement gating
- If your module is a gate: ensure determinism + reason codes
- If your module depends on a gate: refuse when upstream gate artifacts are missing/invalid

### Step 4 — Implement observability
- Correlation + lineage
- Minimal event set (start/finish/outcome)
- Artifact produced events with IDs/hashes

### Step 5 — Conformance tests
- Schema validation tests
- Determinism tests (where required)
- Fail-closed tests (verification/activation)
- “No-new-facts” trace tests (render)
- “No canon mutation” tests (all modules)

### Step 6 — Documentation updates
- Node doc update: responsibilities + I/O + failure modes
- Artifact docs + schemas (if new/changed)
- Protocol doc update (if boundary changes)
- ADR entry if invariants/contracts/gates changed

---

## 9) Common integration patterns

### 9.1 New extractor (Inputs → Claim-IR)
- Inputs: input snapshots + blueprint
- Output: Claim-IR (schema-constrained)
- Rules: preserve provenance, do not collapse ambiguity, do not claim canon

### 9.2 New resolver (Claim-IR → Resolved Claim-IR)
- Inputs: Claim-IR + pinned policies
- Output: Resolved Claim-IR
- Rules: preserve ambiguity, no new facts, record resolution method

### 9.3 New Runtime Pack projection (Kristal → Pack files)
- Inputs: validated canon + pinned config
- Output: new projection in `runtime-pack-manifest`
- Rules: deterministic build, declared capabilities, reproducible manifests

### 9.4 New renderer (Exchange/Pack → Render Bundle)
- Inputs: pinned exchange/pack + template/version + params
- Output: Render Bundle + trace_map
- Rules: no new facts, trace coverage, deterministic mode supported

### 9.5 New execution tool (Orgo Task → SwarmCraft)
- Inputs: Orgo Task contract
- Output: telemetry + artifacts or deterministic failure
- Rules: obey constraints, idempotency strategy, no canon writes

### 9.6 New distribution channel (Konnaxion)
- Inputs: Runtime Pack + manifest + signatures/hashes
- Output: activated pack pointer
- Rules: verify before activate, rollback deterministic, offline-safe trust roots

---

## 10) Checklists

### Producer checklist (writes artifacts)
- [ ] Schema defined + stored in `schemas/`
- [ ] Example payload added
- [ ] Deterministic ordering rules stated
- [ ] Error codes defined
- [ ] Observability events implemented
- [ ] Conformance tests implemented
- [ ] No canon mutation proven

### Consumer checklist (reads artifacts)
- [ ] Supported versions declared
- [ ] Verification behavior specified (fail-closed where required)
- [ ] Missing/unknown fields behavior specified
- [ ] Observability events implemented

### Change checklist (modifies invariants/contracts/gates)
- [ ] ADR created
- [ ] Schema versions bumped
- [ ] Compatibility policy updated
- [ ] Migration guidance added
- [ ] Conformance suite updated

---

## 11) Minimal template for a new integration (copy/paste)

**Name**:  
**Type**: producer | consumer | both  
**Node**:  
**Protocols**:  
**Consumes**: artifact types + versions  
**Produces**: artifact types + versions  
**Determinism**: required | optional (auditable)  
**Gates**: refusal conditions + error codes  
**Security**: confidentiality + verification + least privilege  
**Observability**: required events + fields  
**Conformance tests**: list  
**Docs updated**: files list
