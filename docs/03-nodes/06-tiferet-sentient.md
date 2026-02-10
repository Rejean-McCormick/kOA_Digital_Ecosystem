# 06 — Tiferet: SenTient (Resolution Engine)

## Purpose
**SenTient** is the ecosystem’s **resolution and reconciliation engine**. It converts *proposed* claims (Claim-IR) into **explicit, typed, auditable resolution outputs** (Resolved Claim-IR), while preserving uncertainty and enforcing deterministic structure.

SenTient is the “meaning distillation” step: it does not invent truth; it **binds surfaces to candidates** and **normalizes literals** under strict contracts so downstream gates (validation, compilation) remain reproducible.

> Naming note: “Tiferet” is used as a semantic label for this role in the node map; the operational definition is the contract below.

---

## Responsibilities
SenTient MUST:
- Resolve **entity surfaces** to ranked QID candidates.
- Resolve **property surfaces** to ranked PID candidates.
- Normalize **literal values** into typed forms (dates, quantities, coords, identifiers), recording any lossy steps as warnings.
- Preserve ambiguity explicitly (no silent coercion).
- Produce **schema-valid** outputs with stable, deterministic structure.
- Emit structured warnings/errors that are stable across minor versions.
- Return a syntactically valid output batch even under partial failure (timeouts, upstream outages), with explicit error states.

SenTient MUST NOT:
- Force a single binding when evidence is insufficient.
- Drop claims silently.
- Introduce new claims beyond what Claim-IR proposes.
- Mutate canonical truth artifacts (Exchange / Runtime Packs).
- Act as the validation gate or compiler.

---

## Contract boundary
**Upstream (producer → SenTient):**
- Extractors deliver **Claim-IR** only (schema-constrained proposals + evidence + uncertainty).

**Downstream (SenTient → consumer):**
- SenTient outputs **Resolved Claim-IR** only (explicit resolution states + candidates + typed literals + warnings/errors).

This boundary is strict: it’s where “proposal” becomes “resolution structure” but not yet “canonical truth”.

---

## Inputs

### Primary input: Claim-IR batch
Minimum required per claim:
- `claim_id` (stable)
- surfaces (entity/property/value targets)
- evidence pointers (doc/snippet/dataset refs)
- uncertainty representation (confidence/distribution/modality as defined by Claim-IR)

### Optional inputs: hints and constraints (advisory)
Examples:
- preferred locale/language
- allowed/denied candidate sets
- tenant overlays / mapping constraints
- prior resolution state (incremental resolution)

If hints are used, SenTient MUST record:
- `hints_used: true`
- `hint_refs[]` (references to hint payloads)

---

## Outputs

### Primary output: Resolved Claim-IR batch
For each resolvable surface, SenTient MUST provide:

#### A) Ranked candidates
- `candidates[]` sorted by decreasing score
- each candidate includes:
  - `id` (QID or PID)
  - `score` (numeric, comparable)
  - `evidence` pointers or justification refs (lightweight)
  - optional `features` (informative telemetry, not required for correctness)

#### B) Decision state (explicit)
One of:
- `RESOLVED_SINGLE`
- `RESOLVED_MULTI`
- `UNRESOLVED`
- `ERROR`

#### C) Selected binding (only when resolved)
- If `RESOLVED_SINGLE`, output `selected.id` and `selected.score`.
- If `RESOLVED_MULTI`, `selected` MUST be absent; `candidates[]` MUST remain present (at least top-K).

### Required output metadata
Resolved outputs MUST include sufficient metadata for auditability:
- `resolution_run_id` (unique)
- `sentient_version`
- `timestamp` (ISO 8601)
- `input_batch_ref` (content-addressed reference to the Claim-IR batch)
- `hints_used` + `hint_refs[]` (if applicable)
- `policies` including at minimum:
  - `candidate_top_k` per surface type
  - `tie_breakers`
  - `normalization_ruleset_id`

---

## Determinism requirements (non-negotiable)

### Deterministic structure
SenTient output MUST be deterministic in structure:
- stable field names and types
- stable candidate sorting rules
- stable serialization order once canonicalized downstream

Scores may change between versions; **the structure must not.**

### Top-K truncation policy
If candidates are truncated:
- K MUST be declared in metadata
- truncation MUST be consistent by surface type

### Stable sorting / tie-breakers
Tie-breakers MUST be deterministic:
1) score descending  
2) stable ID ordering (lexicographic QID/PID)  
3) deterministic hash of `(surface, candidate_id)` if needed

### Literal normalization determinism
When normalizing literals, SenTient MUST:
- produce typed outputs in a deterministic schema form
- include explicit unit/precision where relevant
- emit warnings for lossy normalization

---

## Ambiguity preservation
Ambiguity is a first-class output condition, not a bug.

SenTient MUST:
- represent multiple plausible bindings as `RESOLVED_MULTI`
- retain a ranked candidate list rather than collapsing to a single ID
- make unresolved or weak grounding explicit via warnings/errors

---

## Warnings and errors

### Format (stable)
Each warning/error MUST include:
- `code` (machine-readable, stable)
- `severity` (`INFO | WARNING | ERROR`)
- `message` (human-readable)
- optional `details`
- linkage to `claim_id` (and surface identifier where applicable)

### Minimum recommended code families
- `SENTIENT_UNRESOLVED_SURFACE`
- `SENTIENT_AMBIGUOUS_MULTI`
- `SENTIENT_NORMALIZATION_LOSS`
- `SENTIENT_EVIDENCE_INSUFFICIENT`
- `SENTIENT_RESOURCE_LIMIT_TIMEOUT`
- `SENTIENT_UPSTREAM_OUTAGE`
- `SENTIENT_INTERNAL_EXCEPTION`
- `SENTIENT_SCHEMA_INVALID_INPUT` (input rejection boundary)

### Failure behavior (mandatory)
If SenTient cannot complete resolution for some claims due to transient issues:
- MUST return a syntactically valid Resolved Claim-IR batch
- MUST mark affected surfaces as `UNRESOLVED` or `ERROR`
- MUST emit explicit error codes
- MUST NOT silently drop claims

---

## Interface surface (implementation-agnostic)

### Core operation
- `ResolveClaimBatch(claim_ir_batch, hints?, constraints?, policy_id?) -> resolved_claim_ir_batch`

### Operational expectations
- Requests and results MUST support correlation IDs:
  - `resolution_run_id`
  - `claim_id`
  - optional `surface_id`
- The interface MAY be synchronous (bounded) or async (job-based), but the output contract remains identical.

---

## Implementation profile (informative, non-normative)
A reference SenTient implementation can follow a “funnel” strategy:
1) **Ingestion & fingerprinting** (dedupe, cache keys)
2) **Fast candidate generation** (broad recall)
3) **Semantic scoring / re-ranking** (contextual precision)
4) **Judgment / consensus** (final state + telemetry)
5) **Hybrid memory** (keep hot state small; offload heavy payloads)

This section is descriptive only: alternative internal architectures are valid if they satisfy the contract.

---

## Observability (recommended)
SenTient SHOULD emit:
- counters: claims processed, resolved single, resolved multi, unresolved, error
- histograms: latency per stage, per surface type
- distributions: confidence scores per batch
- structured logs keyed by `resolution_run_id` and `claim_id`
- resource-limit events (timeouts, truncations)

---

## Minimal example (Resolved Claim-IR surface)
```json
{
  "claim_id": "c-001",
  "surfaces": [
    {
      "surface_id": "s-entity-1",
      "kind": "ENTITY",
      "text": "Paris",
      "state": "RESOLVED_MULTI",
      "candidates": [
        { "id": "Q90", "score": 0.62, "evidence": ["ev-12"] },
        { "id": "Q47796", "score": 0.60, "evidence": ["ev-12"] }
      ],
      "warnings": [
        {
          "code": "SENTIENT_AMBIGUOUS_MULTI",
          "severity": "WARNING",
          "message": "Multiple candidates remain plausible; no forced disambiguation."
        }
      ]
    }
  ],
  "metadata": {
    "resolution_run_id": "run-2026-02-09T10:12:55Z-0007",
    "sentient_version": "sentient.v1",
    "timestamp": "2026-02-09T10:12:55Z",
    "input_batch_ref": "sha256:…",
    "hints_used": false,
    "policies": {
      "candidate_top_k": { "ENTITY": 20, "PROPERTY": 10 },
      "tie_breakers": ["score_desc", "id_lex", "hash(surface,id)"],
      "normalization_ruleset_id": "norm.v1"
    }
  }
}
````

---

## Integration checklist

SenTient is conformant at the ecosystem boundary if it:

* accepts Claim-IR and rejects schema-invalid input
* emits Resolved Claim-IR that is schema-valid and deterministic in structure
* preserves ambiguity explicitly (no silent coercion)
* never drops claims, even on partial failure
* emits stable, structured warning/error codes
* records policy + hint usage metadata for auditability


