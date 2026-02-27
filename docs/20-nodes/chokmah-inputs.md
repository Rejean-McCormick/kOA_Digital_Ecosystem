# Chokmah — Inputs (Ingest + Provenance Boundary)

**File:** `docs/20-nodes/chokmah-inputs.md`  
**Normative scope:** kOA node behavior and interfaces.  
**Non-normative:** Kristal artifact schemas (external).

---

## 1) Purpose

Chokmah is the ecosystem’s **ingest and provenance boundary**. It turns raw external inputs into **immutable, content-addressed snapshots** that downstream stages can reference reproducibly (extraction, resolution, validation, compilation).

Chokmah does **not** decide truth. It guarantees only that **“what we saw”** is captured, auditable, and replayable.

---

## 2) Responsibilities

Chokmah MUST:

- Ingest raw materials from configured sources (files, feeds, APIs, user submissions).
- Produce **immutable Input Snapshots** stored in a content-addressed store.
- Capture and persist **provenance** for every snapshot (source identity, retrieval time, auth/context, routing tags, policy tags).
- Enforce confidentiality and access control on ingested materials (PII, restricted sources), including encryption at rest when required.
- Provide **idempotent ingestion** (same bytes → same snapshot ref; safe retries).
- Emit a deterministic, machine-readable **input set** reference for downstream builds (recorded in Orgo Build Record).
- Emit deterministic ingestion results (success/warnings/failures) with stable error codes.

Chokmah MAY:

- Normalize **transport/container** format **without changing meaning** (e.g., decoding, decompressing, charset normalization), **only** when:
  - the transform is deterministic, and
  - the transform is recorded explicitly in the snapshot manifest (tool id/version + steps).

Chokmah MUST NOT:

- Interpret, enrich, or “fix” content in a way that changes meaning without recording it explicitly as a derived artifact.
- Generate or alter canonical truth artifacts.

---

## 3) Interfaces

### 3.1 Inputs to Chokmah
- **Ingest Request** (from Orgo or an ingestion orchestrator)
  - Source descriptors (URI, connector type, credentials reference)
  - Expected content type and size (if known)
  - Confidentiality classification / handling constraints
  - Applicable mandate/policy tags (from Keter)
  - Optional: retention/quarantine directives

### 3.2 Outputs from Chokmah
- **Input Snapshot Set**
  - One or more snapshot refs (content-addressed)
- **Snapshot Manifest**
  - Deterministic mapping from snapshot ref → provenance + acquisition metadata
  - Ingestion policy version/ref applied
  - Transformation steps (if any), with deterministic tooling identifiers
  - Integrity metadata (hashes/checksums)
- **Ingest Receipt**
  - Snapshot refs created/confirmed
  - Errors and partial results (if any)
  - Retrieval diagnostics pointers
- Optional: **Quarantine Report**
  - When content is blocked/sanitized/quarantined per policy

> Field-level schemas for kOA-owned operational artifacts live under `docs/30-artifacts/`. If you maintain a dedicated Input Snapshot / Snapshot Manifest schema, treat it as kOA-owned.

---

## 4) Snapshot identity model

- Snapshot identity is **content-addressed**:
  - Same payload bytes MUST yield the same snapshot reference.
- **Metadata changes MUST NOT change the payload reference**; they belong in snapshot metadata/manifest records.
- If the ingest source is mutable (e.g., `latest.json`), Chokmah MUST still store the retrieved bytes as an immutable snapshot and record the retrieval context.

---

## 5) Invariants (normative)

1. **Immutability**
   - Once a snapshot reference is issued, the referenced bytes MUST never change.

2. **Content addressing**
   - Snapshot references MUST be derived from content (and any explicitly defined deterministic packaging rules for stored bytes).

3. **Complete provenance**
   - Every snapshot MUST have provenance sufficient for audit and reproduction:
     - where it came from
     - when it was fetched
     - under what policy/authority context
     - what transformations were applied (ideally none)

4. **Confidentiality enforcement**
   - Restricted snapshots MUST be encrypted at rest and access-controlled.
   - Downstream stages SHOULD receive only references unless explicitly authorized to fetch bytes.

5. **Idempotent ingestion**
   - Re-ingesting the same payload MUST return the same reference.
   - Retries MUST NOT create duplicate “distinct” snapshots for identical bytes.

6. **Build reproducibility**
   - Orgo MUST be able to bind a build to a stable set of snapshot refs (“input set”).
   - Downstream compilation MUST NOT depend on live sources—only recorded snapshot refs.

7. **No hidden enrichment**
   - Ingestion MUST NOT introduce new facts; it only captures and packages input material.

---

## 6) Error handling and failure modes

Chokmah MUST fail closed when it cannot guarantee correctness.

Common failures:
- Source unreachable / authentication failure
- Partial download or truncated payload
- Hash mismatch during streaming verification
- Policy violation (attempt to ingest disallowed source / classification mismatch)
- Quarantine triggered (malware, sensitive data, unsafe content)
- Unsupported format / decoding failure
- Storage failure (cannot commit snapshot immutably)
- Malformed content violating declared type constraints (record as ingest error; do not coerce)

Chokmah SHOULD return structured errors stable across minor versions:
- `code` (stable)
- `message` (human)
- `diagnostics_ref` (pointer)

---

## 7) Observability

Chokmah SHOULD emit:

### 7.1 Metrics
- ingest requests, successes/failures
- bytes ingested, throughput
- latency per connector/source type
- retry counts and idempotency hits
- rejection/quarantine rates and reasons

### 7.2 Logs (structured)
- request id, source descriptor hash, snapshot refs
- policy/classification decisions
- failure codes and diagnostics pointers

### 7.3 Audit events
- snapshot created/confirmed
- access grants/denials
- retention/expiration actions

---

## 8) Security and trust boundaries

- Chokmah touches untrusted external data; treat all inputs as untrusted until committed immutably.
- Connector credentials MUST be handled via a secure secret mechanism; never embedded in artifacts.
- Confidentiality policy MUST be enforced consistently with Keter mandate and kOA operations policy.

---

## 9) Related docs

- `docs/10-system/trust-boundaries.md`
- `docs/50-operations/pipeline.md`
- `docs/30-artifacts/build-record.md`
- `docs/30-artifacts/mandate-bundle.md`
- `docs/40-integration/kristal-v4/contract-pointers.md`