# Chokmah (Inputs)

Chokmah is the system’s **ingest + provenance boundary**. It captures raw external inputs and turns them into **immutable, content-addressed snapshots** so downstream stages can be reproduced and audited.

Chokmah does **not** decide truth. It guarantees only that “what we saw” is captured, traceable, and replayable.

---

## Purpose

- Provide a safe, repeatable ingest boundary for raw data (files, feeds, APIs, user submissions).
- Produce stable snapshot references that the rest of the pipeline can pin to a build.

---

## Responsibilities

Chokmah **must**:

- Ingest raw materials from configured sources.
- Produce **immutable Input Snapshots** stored in a content-addressed store.
- Capture and persist **provenance** per snapshot (source identity, retrieval time, auth/context, routing tags, policy tags).
- Enforce confidentiality and access control (including encryption at rest when required).
- Provide **idempotent ingestion** (same bytes → same snapshot reference).
- Emit a deterministic, machine-readable **input set** reference for downstream builds.
- Emit deterministic ingestion results with stable error codes.

Chokmah **may**:

- Normalize transport/container formats **without changing meaning** (e.g., decode/decompress/charset normalization) only if:
  - the transform is deterministic, and
  - the transform is recorded explicitly in the snapshot manifest (tool id/version + steps).

Chokmah **must not**:

- Interpret, enrich, or “fix” content in a way that changes meaning without recording it as a derived artifact.
- Generate or alter canonical truth artifacts.

---

## Interfaces

### Inputs to Chokmah

**Ingest Request** (from Orgo or an ingestion orchestrator), including:

- Source descriptor (URI / connector type / credentials reference)
- Expected content type and size (if known)
- Confidentiality classification / handling constraints
- Mandate/policy tags
- Optional retention/quarantine directives

### Outputs from Chokmah

- **Input Snapshot Set**: one or more content-addressed snapshot references
- **Snapshot Manifest**:
  - deterministic mapping of snapshot ref → provenance + acquisition metadata
  - ingestion policy version/ref applied
  - transformation steps (if any) with deterministic tooling identifiers
  - integrity metadata (hashes/checksums)
- **Ingest Receipt**:
  - snapshot refs created/confirmed
  - errors/partial results (if any)
  - diagnostics pointers
- Optional **Quarantine Report** (when content is blocked/sanitized/quarantined per policy)

---

## Snapshot identity model

- Snapshot identity is **content-addressed**: same payload bytes → same snapshot reference.
- Metadata changes do **not** change the payload reference; metadata lives in the manifest/records.
- If the source is mutable (e.g., “latest.json”), Chokmah still stores the retrieved bytes as an immutable snapshot and records retrieval context.

---

## Invariants

1) **Immutability**: once issued, snapshot bytes never change.  
2) **Content addressing**: refs derive from content (and any explicitly defined deterministic packaging rules).  
3) **Complete provenance**: every snapshot records where/when/how it was fetched, under what policy/authority context, and any transformations applied.  
4) **Confidentiality enforcement**: restricted snapshots are encrypted at rest and access-controlled; downstream should receive only refs unless explicitly authorized to fetch bytes.  
5) **Idempotent ingestion**: re-ingesting identical bytes returns the same ref; retries don’t create duplicates.  
6) **Build reproducibility**: Orgo can bind a build to a stable snapshot set; downstream must not depend on live sources.  
7) **No hidden enrichment**: ingest captures and packages input material; it does not introduce new facts.

---

## Error handling (fail closed)

Chokmah fails closed when it cannot guarantee correctness. Common failures include:

- Source unreachable / authentication failure
- Partial download or truncated payload
- Hash mismatch during streaming verification
- Policy violation (disallowed source / classification mismatch)
- Quarantine triggered (malware, sensitive data, unsafe content)
- Unsupported format / decoding failure
- Storage failure (cannot commit snapshot immutably)
- Malformed content violating declared type constraints (record as ingest error; do not coerce)

Errors should be structured and stable:

- `code` (stable)
- `message` (human)
- `diagnostics_ref` (pointer)

---

## Observability

Chokmah should emit:

### Metrics
- ingest requests, successes/failures
- bytes ingested, throughput
- latency per connector/source type
- retry counts and idempotency hits
- rejection/quarantine rates and reasons

### Structured logs
- request id, source descriptor hash, snapshot refs
- policy/classification decisions
- failure codes and diagnostics pointers

### Audit events
- snapshot created/confirmed
- access grants/denials
- retention/expiration actions

---

## Security and trust boundary notes

- Chokmah touches untrusted external data; treat all inputs as untrusted until committed immutably.
- Connector credentials are handled via a secure secret mechanism (never embedded in artifacts).
- Confidentiality policy is enforced consistently with mandate and operational policy.

---

## Related pages

- Components-Orgo.md
- Lifecycle.md
- Operations-Builds.md
- Artifacts-Operational.md