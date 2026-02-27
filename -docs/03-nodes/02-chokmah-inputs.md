# Chokmah (Inputs) — Raw Substance / “What”

## Status
Normative node specification.

## Purpose
Chokmah is the ecosystem’s raw intake: it receives unrefined content and assets and makes them available downstream without interpreting meaning or declaring truth.

## Responsibilities
- Ingest raw sources (corpora, feeds, uploads, human-provided material).
- Preserve provenance and ensure inputs remain referenceable and reproducible.
- Provide source snapshots / ingestion batches for downstream structuring and extraction.
- Perform only trivial hygiene transformations (e.g., exact deduplication, basic normalization) when explicitly allowed.

## Non-responsibilities
- No semantic judgment, filtering-by-truth, or validation of claims.
- No disambiguation or resolution.
- No mutation of sources “in place” (downstream consumes copies or references).

## Contract boundaries
### Upstream
- External sources: file repositories, databases, APIs, media stores, human creators.

### Downstream
- Binah (structuring: schemas/organization of content).
- Orgo Ingest stage (workflow-bound ingestion with snapshot IDs and traceability).

### Directionality
- One-way intake: later stages must not write back to Chokmah; feedback becomes new governed work rather than editing raw inputs.

## Inputs
- Raw documents (text, PDFs, transcripts), datasets, media files, notes/ideas.
- Source metadata (origin, timestamps, author/source identifiers, access constraints).
- Optional ingestion configuration (allowed normalization/dedup rules, snapshot policy).

## Outputs
- `input_snapshots[]`: immutable references (content-addressed where possible) to all source materials used in a build.
- `provenance_ids[]`: stable identifiers linking each snapshot to its origin.
- Optional: `fingerprints[]` for deduplication/short-circuiting downstream work.

## Artifacts
- Raw content collections (dataset dumps, document collections, transcripts, creative briefs).
- Ingestion batches and snapshot manifests.
- Optional cache of normalized text keyed by fingerprints (optimization only).

## Invariants (must hold)
- Preserve provenance end-to-end.
- Do not mutate source materials.
- Do not declare or coerce truth at this stage.
- Reproducibility: the same external input should yield the same forwarded content/snapshot references (subject to configured fingerprinting rules).

## Failure modes and handling
- Corrupt/poison inputs (unparseable PDFs, malformed encodings, pathological files):
  - Quarantine and route to DLQ/triage rather than infinite retry or silent drop.
- Duplication storms:
  - Fingerprint + exact dedup to reduce downstream load (no semantic filtering).
- Access/permission failures:
  - Fail clearly with structured error codes; never “best-effort” partial truthing.

## Observability (recommended)
Emit structured events for:
- `tenant_id`, `build_id`, `stage="ingest"`, `input_snapshot_id`, `source_uri` (or redacted token), hash, size, parser outcome.
- Error codes for quarantine/DLQ, with bounded error messages.

## Security and access considerations
- Chokmah may store sensitive raw materials: enforce access control at rest and in transit.
- Provenance identifiers should not leak secrets (use indirection/redaction where needed).
- Integrity is enforced downstream (validation/compile), but snapshot hashing is recommended for traceability.

## Open questions (tracked explicitly)
- Standard snapshot manifest schema (fields, hashing strategy, redaction rules).
- Whether fingerprints are core or profile-specific (cost vs benefit).
