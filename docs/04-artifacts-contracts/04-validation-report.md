# Validation Report (Artifact Contract)

## Status
Normative (required for pipeline gating).

## Purpose
A **Validation Report** is the deterministic, typed output of the **Validation** stage. It exists to:
- provide a machine- and human-readable explanation of acceptance/rejection,
- act as the **compile gate** (“no compile on fail”),
- make validation outcomes auditable, comparable across toolchains, and traceable to inputs/builds.

## Producers / Consumers
**Produced by**
- Kristal Validator (core validation)
- Optional profile validators (e.g., SHACL, ShEx) that append structured results

**Consumed by**
- Orgo (stage gating + operator UX)
- Compiler / build system (hard gate)
- CI / conformance tests
- Observability / audit trails

## Emission rules
- A report MUST be emitted for every validation attempt (pass/fail/partial).
- If `validation_status = "failed"`, the pipeline MUST NOT proceed to compilation/publish.
- If `validation_status = "partial"`, compilation MAY be allowed only if policy explicitly permits it (see `gate.compile_allowed`).

## Canonical format
- **Format:** JSON
- **Schema:** `docs/04-artifacts-contracts/schemas/validation-report.schema.json`
- **Schema version:** `schema_version = "3.0"`

The schema is authoritative for field names and required/optional fields; this document defines the semantics and determinism constraints.

---

## 1) Top-level fields (contract)

### Required
- `schema_version` *(string, const "3.0")*
- `validation_status` *(enum: "passed" | "partial" | "failed")*
- `issues` *(array; can be empty)*
- `gate` *(object; deterministic gate decision)*

### Strongly recommended (traceability + reproducibility)
- `report_id` *(string; UUID or equivalent unique ID)*
- `created_at` *(date-time; UTC)*
- `correlation` *(object: build/workflow/case/task/request IDs)*
- `policy` *(object: policy_id + blueprint_hash + validator mode)*
- `tooling` *(object; validator identity/version/config hash/canonicalization profile)*
- `inputs` *(object; pinned input references + lineage pointers)*

### Optional (profile outputs + extensions)
- `shacl` *(object; report pointers/format when SHACL profile enabled)*
- `shex` *(object; report pointers/format when ShEx profile enabled)*
- `export_validation` *(object; JSON-LD/RDF/WDQS compatibility checks; optional rdf_hash)*
- `extensions` *(object; implementation-specific data; MUST NOT affect gate semantics)*

---

## 2) `validation_status` semantics

- **passed**: no blocking issues; compilation is allowed.
- **partial**: non-blocking findings exist (typically warnings/info). Policy decides whether partial blocks compilation.
- **failed**: blocking issues exist OR required checks could not be performed deterministically (fail-closed); compilation is forbidden.

Status MUST be derived deterministically from `issues[]` under the declared policy (see `policy.*`).

---

## 3) Gate decision (normative)

`gate` is the machine-readable enforcement payload. Minimum:

- `gate.compile_allowed` *(boolean)*: MUST be `false` when `validation_status="failed"`.
- `gate.reason` *(string)*: deterministic category for why compilation is allowed/blocked.
- `gate.policy_mode` *(string)*: e.g., `strict`, `standard`, `lenient` (naming policy-defined, but deterministic).

Recommended categories for `gate.reason`:
- `OK`
- `FAILED_BLOCKING_ISSUES`
- `FAILED_FAIL_CLOSED`
- `PARTIAL_ALLOWED_BY_POLICY`
- `PARTIAL_BLOCKED_BY_POLICY`

---

## 4) Determinism requirements

The Validation Report is part of the determinism surface:
- Given identical pinned inputs, enabled profiles, tooling versions/config hash, and declared policies, the report outcome and issue set MUST be stable.
- Issue ordering MUST be deterministic.

**Recommended ordering (tie-breakers)**
1) severity rank: `ERROR` > `WARNING` > `INFO`  
2) `code` (lexicographic)  
3) `location.json_pointer` (lexicographic)  
4) `location.entity_id` (lexicographic)  
5) stable hash of `(code, message, location)` if needed  

---

## 5) Inputs and traceability (recommended structure)

`inputs` SHOULD include pinned references sufficient to reproduce the validation:

- `claim_ir_ref` *(optional)*: content ref for Claim-IR batch
- `resolved_claim_ir_ref` *(recommended)*: content ref for Resolved Claim-IR batch validated by this run
- `snapshot_set_ref` *(optional)*: content ref to snapshot manifest / input set recipe
- `exchange_candidate_ref` *(optional)*: if validating a compiled candidate exchange payload prior to publish
- `build_record_ref` *(optional)*: if available at validation time
- `exchange_manifest_ref` *(optional)*: if validation covers manifest integrity constraints

If the system uses content-addressed refs, each `*_ref` SHOULD carry at least `{ref, hash, hash_alg}`.

---

## 6) Issues model

`issues[]` is a list of structured findings.

Each issue SHOULD include:
- `severity` *(ERROR | WARNING | INFO)*
- `code` *(stable, machine-readable)*
- `message` *(human-readable explanation)*
- `location` *(object; see below)*
- `details` *(object; optional structured context for tooling)*

### Location model (`issue_location`)
A location provides stable pointers into artifacts and/or identifiers:
- `json_pointer` *(RFC 6901 pointer into Claim-IR/Resolved Claim-IR/Exchange/manifest structures)*
- `entity_id` *(QID/PID/LID/etc or local placeholder)*
- `statement_id` *(string; Exchange record/statement id when applicable)*
- `claim_id` *(string; Claim-IR / Resolved Claim-IR claim id)*
- `evidence_id` *(string; evidence pointer id)*
- `source_uri` *(uri; original provenance URI or snapshot URI when safe)*

Location fields are optional, but at least one stable pointer SHOULD be present for `ERROR` issues.

---

## 7) Codes (stability requirement)

Codes MUST be stable and machine-readable. Recommended naming conventions:
- `VAL.SCHEMA.*` (schema / structural violations)
- `VAL.RESOLUTION.*` (unresolved IDs, invalid entity types, illegal merges)
- `VAL.EVIDENCE.*` (missing/invalid evidence refs, evidence policy violations)
- `VAL.CONSTRAINT.*` (domain constraints)
- `VAL.INTEGRITY.*` (hash/signature/manifest integrity failures; fail-closed)
- `VAL.PROFILE.SHACL.*` / `VAL.PROFILE.SHEX.*` (profile validator findings)

---

## 8) Optional profile sections

### `shacl`
When enabled, the report SHOULD contain:
- `enabled` *(boolean)*
- `report_uri` *(uri)*
- `report_format` *(e.g., "application/ld+json", "text/turtle")*

### `shex`
When enabled:
- `enabled` *(boolean)*
- `report_uri` *(uri)*
- `report_format` *(e.g., "application/json", "text/plain")*

### `export_validation`
Optional checks for deterministic exports:
- `jsonld_profile` *(string)*
- `rdf_profile` *(string)*
- `wdqs_compat` *(boolean)*
- `rdf_hash` *(string; only when an RDF integrity profile is enabled)*

---

## 9) Minimal examples

### Passed (minimal)
```json
{
  "schema_version": "3.0",
  "validation_status": "passed",
  "issues": [],
  "gate": {
    "compile_allowed": true,
    "reason": "OK",
    "policy_mode": "strict"
  },
  "report_id": "9b7b2a76-5b88-4e4a-9f6e-0f58bfe6f1d2",
  "created_at": "2026-02-09T00:00:00Z",
  "correlation": {
    "build_id": "build_2026-02-09_000001"
  },
  "tooling": {
    "validator_name": "kristal-validator",
    "validator_version": "3.0.0",
    "config_hash": "sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "canonicalization_profile": "kristal.v3:jcs-rfc8785",
    "canonicalization_version": "1"
  },
  "policy": {
    "policy_id": "policy.core.strict@1",
    "blueprint_hash": "sha256:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  }
}
````

### Failed (typical)

```json
{
  "schema_version": "3.0",
  "validation_status": "failed",
  "issues": [
    {
      "severity": "ERROR",
      "code": "VAL.EVIDENCE.MISSING",
      "message": "Claim has no evidence references; cannot compile into Exchange.",
      "location": {
        "json_pointer": "/claims/12/evidence",
        "claim_id": "claim_12"
      },
      "details": {
        "expected_min_evidence": 1,
        "found": 0
      }
    }
  ],
  "gate": {
    "compile_allowed": false,
    "reason": "FAILED_BLOCKING_ISSUES",
    "policy_mode": "strict"
  },
  "report_id": "f8a18f2c-3f6d-4f7a-a405-16e21ee3b6c7",
  "created_at": "2026-02-09T00:01:00Z",
  "correlation": {
    "build_id": "build_2026-02-09_000002"
  }
}
```

---

## 10) Conformance tests (required)

At minimum:

* validate JSON against `validation-report.schema.json`
* ensure deterministic ordering of `issues[]`
* ensure `validation_status="failed"` halts compilation/publish (“no compile on fail”)
* ensure stable `code` values for the same failure class across runs/toolchains
* ensure `gate.compile_allowed` is consistent with status + policy (deterministic)

