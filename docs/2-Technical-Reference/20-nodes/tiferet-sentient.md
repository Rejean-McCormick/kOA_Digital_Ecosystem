# Tiferet — SenTient (Resolution Engine)

**File:** `docs/20-nodes/tiferet-sentient.md`  
**Normative scope:** kOA node behavior and interfaces.  
**Non-normative:** Kristal artifact schemas (external), cryptographic/canonicalization specifics (external).

---

## 1) Purpose

SenTient is the ecosystem’s **resolution and reconciliation engine**. It converts *proposed* claims (Claim-IR) into **explicit, typed, auditable resolution outputs** (Resolved Claim-IR), while preserving uncertainty and enforcing deterministic structure so downstream gates remain reproducible.

SenTient does not “create truth.” It produces **well-structured candidates and decisions** that can be validated and compiled into canonical truth later.

---

## 2) Responsibilities

SenTient MUST:

- Resolve **entity surfaces** to ranked candidate identifiers.
- Resolve **property surfaces** to ranked candidate identifiers.
- Normalize **literal values** into typed forms (dates, quantities, coordinates, identifiers), recording any lossy steps as warnings.
- Preserve ambiguity explicitly (no silent coercion).
- Produce **schema-valid** Resolved Claim-IR outputs with stable ordering and deterministic structure.
- Emit stable, machine-readable **diagnostics** (warnings/errors) for downstream gating and operator review.
- Maintain traceability from each resolved item back to its originating claim and evidence pointers.

SenTient SHOULD:

- Provide per-domain resolvers (people/orgs/places/medical/finance/etc.) behind a single interface.
- Support “policy-driven resolution” (e.g., stricter normalization rules) without changing the public artifact contract.

---

## 3) Inputs

### 3.1 Required inputs
- **Claim-IR batch** (proposed claims with evidence pointers and extraction context)
- **Resolution policy bundle** (kOA policy/config applicable to this run)
- **Resolver resources** (indexes, dictionaries, embeddings, lookup tables, ontologies), versioned and pinned by Orgo

### 3.2 Optional inputs
- **Context bundles** (domain packs, locale packs, tenant-specific mappings)
- **Prior state hints** (e.g., last-known mappings) as *non-authoritative* hints

---

## 4) Outputs

### 4.1 Primary output
- **Resolved Claim-IR batch**: a deterministic, schema-valid batch where each claim is transformed into:
  - typed normalized literals (where applicable),
  - ranked candidates for entities/properties,
  - explicit resolution state (resolved / ambiguous / rejected / error),
  - diagnostics and provenance pointers.

### 4.2 Secondary outputs
- **Resolution diagnostics report** (optional): aggregated counters and top error categories for observability.
- **Resolver telemetry** (required): structured events for latency, hit-rate, and drift detection (no new facts).

---

## 5) Determinism and invariants

### 5.1 Determinism requirements
SenTient output MUST be deterministic given:
- the exact Claim-IR input,
- pinned resolver resources,
- pinned policy bundle,
- pinned code version.

Determinism means:
- stable ordering of records and candidate lists,
- stable scoring behavior (within defined numerical tolerances),
- stable diagnostic codes.

### 5.2 No silent upgrades
SenTient MUST NOT change:
- normalization rules,
- ranking thresholds,
- candidate set construction

without a versioned change that Orgo can pin and audit.

### 5.3 Ambiguity preservation
If multiple candidates are plausible, SenTient MUST record ambiguity explicitly rather than choosing silently.

### 5.4 No truth mutation
SenTient MUST NOT:
- rewrite or “correct” upstream evidence,
- invent missing facts,
- collapse uncertainty into a single asserted truth.

---

## 6) Interface (Orgo integration)

### 6.1 Invocation
SenTient is invoked by Orgo as a pipeline stage:
- input: content-addressed Claim-IR reference + policy/resource refs
- output: content-addressed Resolved Claim-IR reference + diagnostics refs

### 6.2 Idempotency
Re-running SenTient with identical pinned inputs MUST produce byte-identical (or canonical-identical) outputs.

### 6.3 Failure handling contract
- On **hard failure**, SenTient returns a structured error and produces no Resolved Claim-IR output.
- On **partial resolution**, SenTient still produces Resolved Claim-IR but marks affected claims as ambiguous/rejected with diagnostics.

Orgo decides gating rules; SenTient must provide the signals needed for those decisions.

---

## 7) Error model

SenTient MUST emit stable error/warning codes, including (minimum):

- `RESOLVE_ENTITY_NOT_FOUND`
- `RESOLVE_ENTITY_AMBIGUOUS`
- `RESOLVE_PROPERTY_NOT_FOUND`
- `RESOLVE_PROPERTY_AMBIGUOUS`
- `NORMALIZE_LITERAL_INVALID`
- `NORMALIZE_LITERAL_LOSSY`
- `EVIDENCE_POINTER_INVALID`
- `POLICY_VIOLATION`
- `RESOURCE_VERSION_MISSING`
- `INTERNAL_RESOLVER_ERROR`

Each diagnostic MUST include:
- claim reference
- field/path reference (where applicable)
- severity
- deterministic message template + structured parameters

---

## 8) Observability

SenTient MUST emit:

### 8.1 Metrics
- throughput (claims/sec)
- p50/p95/p99 latency
- candidate hit-rate (entity/property)
- ambiguity rate
- rejection rate
- normalization error rate
- resource cache hit-rate

### 8.2 Logs (structured)
- run_id / build_id (from Orgo)
- input content ref
- pinned policy/resource versions
- top diagnostic codes + counts

### 8.3 Tracing
- stage span per run
- child spans per resolver component (entity/property/literal normalization)

---

## 9) Security and privacy

- SenTient must treat inputs as potentially sensitive and follow tenant policy for logging/redaction.
- Resolver resources must be integrity-checked (hash/signature) and version-pinned by Orgo.
- No external network calls unless explicitly allowed by policy and audited.

---

## 10) Test and conformance expectations (kOA)

Minimum test suite:
- deterministic rerun test (same inputs/resources → same outputs)
- ambiguity preservation test
- literal normalization golden tests (per domain)
- resource pinning test (mismatched resource ref → failure)
- diagnostic stability test (codes and shapes stable across patch releases)

---

## 11) Links

- Node map and lifecycle: `docs/10-system/`
- Orgo pipeline stage contract: `docs/50-operations/pipeline.md`
- Integration profile for Kristal compilation boundaries: `docs/40-integration/kristal-v4/`