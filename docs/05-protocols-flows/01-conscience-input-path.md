# Conscience (Input Trunk Path)

## Summary
Conscience is the input trunk path that brings raw matter into the ecosystem **with provenance** and without premature semantic collapse. Its equilibrium rule is: **do not resolve too early**; preserve diversity and ambiguity until the resolution and validation boundaries.

Conscience is not “truth-making.” It is **matter intake + reconciliation interface** that produces schema-valid atomic units and commands for downstream reconciliation.

---

## Producer → Consumer
- **Producer(s):** external sources, ingestion adapters, user interfaces
- **Consumer:** SenTient (Tiferet) ingestion & reconciliation boundary

---

## Purpose (what this path exists to do)
1) Ingest raw content with provenance.
2) Allow UI→Core reconciliation actions without breaking determinism or traceability.
3) Preserve ambiguity and diversity so resolution happens explicitly (not implicitly).

---

## Payloads (contract boundary objects)

### A) Reconciliation Command Protocol (UI → SenTient)
A command protocol used by UI/clients to ask for:
- ingestion of a source snapshot
- reconciliation/clarification actions
- correction signals / feedback capture (as proposals, not canon edits)

**Constraints**
- Commands MUST be schema-valid.
- Commands MUST carry correlation IDs and provenance references where applicable.
- Commands MUST NOT directly mutate canonical truth.

### B) SmartCell (Atomic Data Contract)
Atomic unit used to transport ingested content and candidate evidence into SenTient.

**Constraints**
- SmartCells MUST preserve provenance.
- SmartCells MUST NOT collapse ambiguity.
- SmartCells MUST be schema-valid and deterministic in structure.

> Note: SmartCell schema is referenced as an internal atomic contract for the Conscience path, but should be standardized as a formal schema artifact.

---

## Directionality
- Predominantly **one-way input** (matter enters the system).
- UI interaction is allowed, but only as **typed commands/proposals**, never as direct canon mutation.

---

## Non-negotiable constraints (hard)
1) **Preserve provenance**
   - Every ingested unit must retain source identity and retrieval context.
2) **Do not collapse ambiguity**
   - Candidate identity resolution happens explicitly in SenTient; Conscience must not silently “pick one.”
3) **Deterministic schema outputs**
   - Payload shape and ordering rules must be stable given the same inputs/config.

---

## Relationship to sub-paths
Conscience is the trunk intake; it includes or precedes the following contract-grade sub-boundaries:

### Sub-path: Écriture (Formulation Boundary)
- **Producer → Consumer:** Extractors → SenTient
- **Payload:** Claim-IR (schema-constrained proposals + evidence/uncertainty)
- **Constraints:** extractors output Claim-IR only; must include evidence pointers + uncertainty; schema-invalid payloads rejected.

Écriture is where raw matter becomes **proposed claims**. Conscience is the broader intake + reconciliation interface that ensures provenance and non-collapse.

---

## What Conscience must NOT do
Conscience must not:
- force disambiguation or coerce ambiguity into a single ID,
- invent claims,
- write directly to canonical truth artifacts (Exchange).

These prohibitions align with the downstream SenTient boundary rules (no silent coercion; no invention beyond what is proposed).

---

## How this path fits in the pipeline
Orgo enforces the global workflow stage order:
1) Ingest
2) Extract → Claim-IR
3) Resolve (SenTient) → Resolved Claim-IR
4) Validate
5) Compile → Exchange
6) Compile → Runtime Pack
7) Publish/Distribute

Conscience corresponds to the **Ingest** stage and the UI reconciliation surface that feeds SenTient without bypassing stage gates.

---

## Minimal data requirements (recommended)
### Provenance (required)
Every ingested unit SHOULD provide:
- `source_url` or `source_id`
- `retrieved_at` (timestamp)
- content hash (recommended)
- optional `published_at` (if known)

### Correlation (required for UI-driven actions)
Commands SHOULD include:
- `request_id`
- `case_id` / `task_id` (if applicable)
- `tenant_id` (if multi-tenant)

---

## Failure modes (recommended stable codes)
- `INPUT_SCHEMA_INVALID` — SmartCell/command fails schema validation
- `PROVENANCE_MISSING` — missing required source identity fields
- `AMBIGUITY_COLLAPSED` — payload incorrectly coerced to a single identity
- `UNSUPPORTED_MEDIA_TYPE` — ingestion adapter cannot parse source
- `DUPLICATE_INGEST_CONFLICT` — conflicting snapshots for same declared identity
- `COMMAND_NOT_AUTHORIZED` — caller cannot issue reconciliation command
- `CORRELATION_MISSING` — UI command missing correlation identifiers

All failures should be deterministic, explicit, and logged with correlation IDs.

---

## Observability (required)
Conscience implementations MUST emit:
- ingestion events (accepted/rejected) with reason codes
- provenance fields (source_id/url, retrieved_at, content hash when available)
- correlation IDs (request_id; case/task/workflow when present)
- payload IDs (smartcell_id / ingest_batch_id if defined)

Recommended metrics:
- ingestion success/failure rate by source type
- provenance completeness rate
- ambiguity-rate (how often multi-candidate structures appear)
- schema rejection rate

---

## Conformance tests (minimum)
1) **Provenance preservation**
   - Ingested unit retains source identity through handoff to SenTient.
2) **Ambiguity preservation**
   - Multi-candidate identity remains multi-candidate; no silent coercion.
3) **Schema enforcement**
   - Schema-invalid commands/units are rejected at the boundary.
4) **No canon mutation**
   - Conscience has no write path to Kristal Exchange/Runtime Pack artifacts.
5) **Deterministic output shape**
   - Given identical inputs/config, emitted payload structure and ordering is stable.

---

## Open Questions / TODO
- Standardize **SmartCell schema** (fields, identity rules, provenance requirements).
- Standardize **UI reconciliation command schema** (commands, permissions, correlation).
- Define deduplication rules for repeated ingestion of the same source (snapshot identity vs document identity).
- Define required media adapters and how parsing errors map to stable codes.
