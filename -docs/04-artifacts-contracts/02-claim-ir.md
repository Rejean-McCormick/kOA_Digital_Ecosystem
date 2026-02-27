# Claim-IR (Claim Intermediate Representation)

## Summary
Claim-IR is the ecosystem’s **raw, schema-constrained intermediate representation** for extracted claims.
It is produced by extractors (LLM or non-LLM) and consumed by SenTient for resolution.

Claim-IR is **not canonical truth**. It is a *proposal layer* that must preserve provenance, uncertainty,
and ambiguity so downstream resolution and validation can operate without hidden coercion.

---

## Contract boundary (“Écriture”)
**Producer → Consumer:** Extractors → SenTient  
**Payload:** Claim-IR (`claim-ir.schema.json`)  
**Constraints:**
- Extractors output **Claim-IR only** (no direct writes to canon).
- Must include **evidence pointers** and **uncertainty**.
- Schema-invalid payloads are rejected at the boundary.  

---

## When to use Claim-IR
Use Claim-IR whenever the system is turning unstructured or semi-structured inputs into structured candidate claims, including:
- document/article ingestion
- transcript parsing
- web snapshots and scraped sources
- user-provided text blobs

---

## Top-level structure (recommended)
A Claim-IR file/batch should contain:

- `schema_version` (string)
- `document` (source metadata; identity + provenance)
- `extraction` (run metadata; producer/model/prompt refs)
- `subject` (surface form + candidate IDs)
- `claims[]` (each claim has predicate + object + evidence + uncertainty)
- optional `warnings[]` (structured codes/messages/paths)

---

## Definitions

### Claim-IR batch
A collection of extracted claims for a single document (or a single extraction run),
including run metadata and shared provenance.

### Claim record
A single extracted assertion about a subject, expressed as:
- predicate reference (candidate property IDs)
- object value (candidate entity IDs or literals)
- qualifiers (optional)
- evidence pointers (quotes, offsets/selectors, source IDs/URLs)
- uncertainty annotation

---

## Required fields and constraints (normative)

### 1) Document provenance (required)
A Claim-IR batch MUST carry enough information to trace back to the source:
- stable document identifier (recommended)
- `source_url` or `source_id`
- retrieval timestamp
- optional publish timestamp
- content hash (recommended)

Rationale: without provenance, the pipeline cannot enforce auditability.

### 2) Extraction metadata (required)
A Claim-IR batch MUST record:
- run identifier
- producer identity (tool name/version)
- model identity (if applicable)
- prompt reference/hash (if applicable)

Rationale: enables determinism audits and reproducibility comparisons.

### 3) Subject candidate set (required)
`subject` MUST include:
- `surface` (the raw mention)
- language
- `candidates[]` (ranked options with IDs and scores)

Extractors MUST NOT collapse ambiguity (e.g., pick one QID silently).
Ambiguity must be represented as multiple candidates.

### 4) Claims array (required)
Each element in `claims[]` MUST include:
- `claim_id` (stable within the batch)
- `predicate` (surface + candidates; IDs where possible)
- `object` (typed value; entity candidate set or literal normalization candidates)
- `evidence[]` (at least one evidence pointer recommended)
- `uncertainty` (confidence and method notes recommended)

Each claim MUST be schema-valid; otherwise the entire payload is rejectable at the boundary.

### 5) Evidence pointers (normative)
Evidence MUST include a `source` object that contains either:
- `source_url`, or
- `source_id`

Evidence SHOULD include:
- `quote` excerpt
- offsets (`start_char`, `end_char`) when available
- optional selector (`css`, `xpath`, `json_pointer`, `pdf_region`) for robust anchoring

### 6) Uncertainty annotation (normative)
Claim-IR SHOULD include uncertainty information such as:
- `confidence` in [0..1]
- optional confidence interval
- method label and notes

Uncertainty is not optional philosophically; if the extractor cannot support a confident mapping,
it must say so rather than forcing a decision.

### 7) Warnings (recommended)
`warnings[]` entries SHOULD exist when:
- the subject mapping is ambiguous
- normalization is needed (dates, units, coordinates, etc.)
- extraction had partial failures

Warnings should be machine-parseable (`code`, `message`, `path`).

---

## Determinism expectations (recommended)
Claim-IR generation should be deterministic in shape and ordering:
- stable sorting rules for candidates
- stable claim_id assignment strategy
- stable warning ordering
- explicit language tags

If an extractor uses probabilistic components, it should record enough metadata to reproduce
(or it should persist intermediate candidate sets).

---

## Validation at the boundary
At the Extractors → SenTient boundary, the system SHOULD enforce:
1) JSON schema validation (`claim-ir.schema.json`)
2) minimal provenance presence (`document.source_url` or `document.doc_id`)
3) evidence presence rules (at least one evidence item or an explicit warning code explaining why missing)
4) uncertainty presence rules (confidence or explicit warning)

Schema invalid → reject.

---

## Downstream expectations (SenTient consumption)
SenTient consumes Claim-IR as the **only** source for extracted assertions.
Resolution must:
- preserve `claim_id` for traceability
- preserve or expand ambiguity explicitly (never silently coerce)
- avoid inventing new claims not present in Claim-IR

---

## Stable error / refusal codes (recommended)
- `CLAIM_IR_SCHEMA_INVALID`
- `MISSING_DOCUMENT_PROVENANCE`
- `MISSING_EVIDENCE_POINTERS`
- `MISSING_UNCERTAINTY`
- `SUBJECT_AMBIGUOUS` (warning)
- `VALUE_NORMALIZATION_NEEDED` (warning)
- `EXTRACTOR_PARTIAL_FAILURE` (warning)

---

## Minimal example (excerpt)
(Use the full example in `10-examples/claim-ir.example.json` as the canonical reference.)

- document: `{ doc_id, title, lang, source_url, retrieved_at, published_at, content_hash }`
- extraction: `{ run_id, producer, model, prompt_ref }`
- subject: `{ surface, lang, candidates[] }`
- claim: `{ claim_id, predicate(candidates[]), object(kind + candidates/literal), evidence[], uncertainty }`

---

## Open Questions / TODO
- Decide strict minimum for evidence cardinality: MUST 1+ evidence per claim vs allow missing with warnings.
- Standardize claim_id format and collision rules across multi-document batches.
- Formalize canonical warning/error enums (shared across extractor implementations).
- Define how Claim-IR relates to SmartCell (if SmartCell is the atomic unit used internally).
