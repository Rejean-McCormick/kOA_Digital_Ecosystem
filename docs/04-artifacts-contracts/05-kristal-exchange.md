# Kristal Exchange (Canonical Truth Artifact)

## Summary
Kristal Exchange (“Exchange”) is the ecosystem’s canonical, auditable, content-addressed **source of truth** for validated knowledge.
Exchange is designed to be **mergeable** and **comparable across toolchains**, while keeping the normative core small and expressing advanced behaviors as explicit profiles.

Exchange is produced only through the governed pipeline (validation → compilation). It is never edited in place.

---

## Position in the pipeline
### Producer → Consumer
- Producer: Kristal compiler (after deterministic validation gate)
- Consumers: Architect, query engines, export pipelines, authorized UIs, distribution pack compilers

### Gate rule (hard)
- Validation is the acceptance gate: if validation fails, compilation MUST stop (“no compile on fail”).

---

## Relationship to other artifacts
A Kristal release outputs:
1) **Kristal Exchange** (this artifact) — canonical truth
2) **Kristal Runtime Pack** — derived offline-executable form

Both artifacts MUST have an associated Manifest.

---

## Definitions

### Exchange payload
The canonical knowledge payload (schema-defined; implementation may store as JSON, JSON-LD, RDF, or another deterministic representation under a declared profile).

### Exchange Manifest
The minimal reproducibility and integrity metadata for the Exchange artifact (required).

### exchange_without_signatures
The Exchange payload with signature material removed deterministically (used for hashing / identity).

---

## Hard invariants (Exchange-level)
1) **Canonical truth boundary**
   - Truth is canonical only after validation + compilation into Exchange.
2) **Immutability**
   - Exchange is immutable once published; updates create a new content-addressed artifact.
3) **Content-addressed identity**
   - Exchange identity is derived from canonicalized bytes of exchange_without_signatures.
4) **Integrity is fail-closed**
   - If integrity material is declared and verification fails, verifiers must fail closed.
5) **Reproducibility**
   - Rebuilds with identical inputs/config must reproduce identical IDs.

---

## Canonicalization and identity (mandatory)

### Canonical JSON
- `canonical_json` MUST be RFC 8785 JSON Canonicalization Scheme (JCS).
- Implementations MUST NOT introduce additional canonicalization steps that change bytes beyond JCS.

### Content-addressed ID
Exchange MUST have a stable `kristal_id` computed as:

`kristal_id = sha256( JCS( exchange_without_signatures ) )`

### Canonicalization declaration
Exchange (and its manifest) MUST declare:
- `canonicalization_profile` (e.g., `jcs-rfc8785`)
- `canonicalization_version` (e.g., `1.0`)

---

## Integrity and signatures (mandatory)

### Signature envelope
If signatures are present, they MUST be in a clearly separated envelope such that:
- signature material can be removed deterministically prior to hashing
- the hashed content is unambiguous

### Hash/sign workflow (normative)
Signing and verification MUST follow this order:
1. Remove signature material (if present)
2. Canonicalize via JCS
3. Hash (SHA-256)
4. Verify and/or attach signatures

### Fail-closed semantics (normative)
If an artifact declares any integrity material (hash, signature, signer identity / key reference),
then verifiers MUST:
- fail closed if verification fails
- fail closed if declared integrity material is malformed or ambiguous
- fail closed if declared hash does not match computed hash

Unknown non-integrity fields MUST be ignored for forward compatibility.

---

## Minimal reproducibility manifest (mandatory)

### Required fields (minimum)
Every Exchange MUST have a manifest containing at minimum:
- `schema_version` (const `3.0`)
- `kristal_id`
- `canonicalization_profile`
- `canonicalization_version`
- `content_hash` (hash of canonicalized Exchange with signatures excluded)
- `build` (build metadata)
- `inputs` (input snapshots and references)

In addition, minimal reproducibility requires manifest content covering:
- `build_id` (unique identifier for the build)
- `build_timestamp` (ISO 8601)
- `compiler.name`, `compiler.version`
- `config_hash` (hash of the full build configuration)
- `input_snapshots` (content-addressed references to inputs used)
- `policy_selections` (portable policy identifiers)
- declared hashes for the artifact payload(s)

### Rebuild determinism requirement (normative)
Given identical:
- input snapshots
- compiler version
- configuration (as defined by `config_hash`)
- policy selections

then Exchange rebuild MUST produce identical `kristal_id`.

---

## Data model (normative minimum vs recommended structure)

### Normative minimum
The Exchange payload MUST:
- be canonicalizable under the declared canonicalization profile
- be hash-identifiable by `kristal_id` rules
- be accompanied by a schema-valid manifest
- preserve enough provenance/evidence linkage to support audit and downstream traceability requirements

### Recommended payload structure (informative)
(Exact schema may be specified in an Exchange payload schema file; this is a recommended minimal shape.)

- `schema_version` (3.0)
- `kristal_id` (optional duplication; canonical source remains manifest)
- `canonicalization_profile`, `canonicalization_version`
- `knowledge`:
  - `statements[]`:
    - `statement_id` (stable within payload)
    - `claim_id` (links back to resolved claim inputs)
    - `subject`, `predicate`, `object` (typed)
    - `qualifiers[]` (optional)
    - `provenance[]` (evidence pointers, source refs)
    - `confidence/uncertainty` (optional; when carried forward)
- `signatures` (optional envelope, excluded from hashing)

---

## Mergeability and comparability (design intent)
Exchange is intended to be:
- mergeable representation of validated knowledge claims
- comparable across toolchains (identical content ⇒ identical IDs when canonicalized)

Implementations SHOULD document merge strategy (e.g., statement identity rules, conflict policy) as a profile, not as unstated behavior.

---

## Read and access patterns (informative)
Exchange is optimized for:
- stable query/export by authorized consumers
- generation of derived Runtime Packs
- deterministic rendering by Architect after validation

Writes are only performed via the formal pipeline (governed change; no direct mutation).

---

## Failure modes (recommended stable codes)
- `EXCHANGE_SCHEMA_INVALID`
- `MANIFEST_SCHEMA_INVALID`
- `CANONICALIZATION_PROFILE_UNKNOWN`
- `HASH_MISMATCH`
- `SIGNATURE_INVALID`
- `REPRODUCIBILITY_SURFACE_INCOMPLETE` (missing required manifest fields)
- `VALIDATION_NOT_PASSED` (attempted compile without passing validation)

---

## Conformance tests (minimum)
1) **Identity test**
   - Removing signatures + JCS + SHA-256 yields the declared `kristal_id`.
2) **Fail-closed integrity**
   - Tamper payload or signature → verification fails closed.
3) **Manifest completeness**
   - Missing required manifest fields → reject as non-conformant.
4) **Rebuild determinism**
   - Same inputs/config/compiler/policies → identical `kristal_id`.
5) **No compile on fail (pipeline conformance)**
   - Validation failure produces no new Exchange artifact.

---

## Open Questions / TODO
- Specify the Exchange payload schema file (separate from exchange-manifest.schema.json).
- Specify a standardized merge profile (identity rules for statements, conflict handling).
- Specify baseline export profiles (JSON-LD / RDF) as optional standardized profiles.
