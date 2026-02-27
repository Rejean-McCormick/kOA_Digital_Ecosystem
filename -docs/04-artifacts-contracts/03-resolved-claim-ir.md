# Resolved Claim-IR (Contract)

Resolved Claim-IR is the canonical **post-resolution** intermediate representation produced by **SenTient**. It is the required input to **Validation (Orgo gate)** and then to **Compilation (Kristal)** when validation passes.

Resolved Claim-IR is still **not canon**. It is a structured, auditable, resolution-aware set of claims with explicit ambiguity handling and strict provenance.

---

## 1) Purpose

Resolved Claim-IR exists to:
- carry forward **only what is justified** from upstream Claim-IR + pinned policies
- make **conflict handling explicit** (resolved vs unresolved vs rejected)
- preserve **ambiguity explicitly** when resolution is not justified
- provide deterministic structure for **validation** and **compilation**

---

## 2) Ownership and lifecycle position

- **Producer**: SenTient (Resolution / Harmonization node)
- **Consumers**:
  - Orgo (Validation gate)
  - Kristal compiler (only after validation PASS)
- **Inputs (pinned)**:
  - Claim-IR batch(es) (content-referenced)
  - Snapshot set / provenance refs (content-referenced, when available)
  - Blueprint hash + resolution policy id/version (pinned)

---

## 3) Normative constraints (MUST / MUST NOT)

### 3.1 Truth discipline (no-new-facts upstream)
- Resolved Claim-IR MUST NOT introduce new factual content that is not traceable to upstream Claim-IR claims and their evidence pointers.
- Resolved Claim-IR MUST preserve ambiguity where a single resolution is not justified.
- Resolved Claim-IR MUST declare an explicit decision state for each claim outcome (resolved / unresolved / rejected / merged) and MUST keep the losing candidates traceable.

### 3.2 Provenance and lineage
- Every emitted resolved claim MUST reference:
  - its source Claim-IR batch references, and
  - the source claim ids, and
  - evidence pointer refs sufficient to support downstream validation and renderer trace maps.
- If a resolved claim results from merging, lineage MUST include all merged source claims.

### 3.3 Determinism
- Given identical pinned inputs and pinned blueprint/policy selections, the Resolved Claim-IR output MUST be reproducible in deterministic mode:
  - stable normalization rules,
  - stable ordering rules,
  - stable ids (or stable id derivation inputs).
- If any stochastic behavior is permitted by policy, it MUST be declared (engine version, config hash, and seed policy/value).

---

## 4) Data model

Resolved Claim-IR is an envelope containing:
- identity + producer metadata,
- pinned input references + policy/blueprint pins,
- a set of resolved claim objects,
- conflict groups (slot-level competition),
- ambiguity groups (render-facing ambiguity structure),
- statistics and optional extensions.

### 4.1 Top-level envelope (minimum)

```json
{
  "type": "resolved_claim_ir",
  "schema_version": "1.0",

  "resolved_claim_ir_id": "rcir_sha256_...",

  "created_at": "2026-02-09T00:00:00Z",

  "producer": {
    "component": "SenTient",
    "version": "x.y.z",
    "build": "build_123",
    "instance_id": "sentient-01"
  },

  "determinism": {
    "mode": "strict",
    "config_hash": "sha256:...",
    "seed_policy": "forbidden"
  },

  "policy": {
    "blueprint_hash": "sha256:...",
    "policy_id": "policy.resolution@1",
    "mandate_id": "mandate_...",
    "mandate_version": "vX"
  },

  "inputs": {
    "claim_ir_refs": [
      { "ref_type": "content_addressed", "ref": "sha256:...", "hash": "...", "hash_algorithm": "sha256" }
    ],
    "snapshot_set_ref": { "ref_type": "content_addressed", "ref": "sha256:...", "hash": "...", "hash_algorithm": "sha256" }
  },

  "claims": [],
  "conflict_groups": [],
  "ambiguity_groups": [],

  "statistics": {
    "claims_total": 0,
    "resolved": 0,
    "unresolved": 0,
    "rejected": 0,
    "merged": 0
  },

  "extensions": {}
}
````

Notes:

* `resolved_claim_ir_id` SHOULD be content-addressed (or derived from content under a declared canonicalization profile).
* `inputs.snapshot_set_ref` MAY be omitted if the system does not bundle snapshot sets at this stage, but any available provenance pins SHOULD be included.

---

## 5) Resolved claim object

Each resolved claim is a normalized assertion with explicit state, lineage, and constraints.

### 5.1 Minimal claim shape

```json
{
  "resolved_claim_id": "rcl_...",

  "triple": {
    "subject": { "id": "Q42", "kind": "entity" },
    "predicate": { "id": "P31", "kind": "property" },
    "object": { "id": "Q5", "kind": "entity" }
  },

  "qualifiers": {
    "time": { "start": null, "end": null },
    "jurisdiction": null,
    "scope": null
  },

  "state": "RESOLVED",

  "resolution": {
    "method": "select|merge|reject|defer",
    "reason_codes": ["SOURCE_CONFLICT"],
    "decision_rules": ["rule_..."],
    "conflict_group_id": "cg_...",
    "confidence": 0.0
  },

  "lineage": {
    "source_claim_ir_refs": ["sha256:..."],
    "source_claim_ids": ["clm_..."],
    "source_evidence_refs": ["ev_..."]
  },

  "provenance": {
    "evidence_pointers": [
      { "snapshot_id": "snap_...", "selector": "doc://...#L10-L20" }
    ]
  },

  "notes": {
    "warnings": [],
    "redactions": []
  }
}
```

### 5.2 State enum (normative)

* `RESOLVED`: a single preferred assertion was selected or a merge produced one preferred assertion.
* `UNRESOLVED`: ambiguity preserved; no single binding exists.
* `REJECTED`: claim(s) invalid / policy-blocked / insufficient evidence; not eligible for compilation.
* `MERGED`: administrative marker indicating consolidation occurred (the effective truth-state is still reflected via `state` and conflict/ambiguity groups).

Rule:

* If `state="RESOLVED"`, there MUST exist a justification trail (`resolution.reason_codes` and/or `resolution.decision_rules`) and complete lineage/evidence refs.
* If `state="UNRESOLVED"`, the claim MUST be associated with an ambiguity structure (via `resolution.conflict_group_id` and/or membership in `ambiguity_groups[]`) so downstream renderers can preserve ambiguity.

---

## 6) Conflict groups (slot competition)

A conflict group bundles competing candidates for the same slot (typically same subject+predicate with differing objects/qualifiers).

```json
{
  "conflict_group_id": "cg_...",

  "slot": {
    "subject_id": "Q42",
    "predicate_id": "P569",
    "qualifier_fingerprint": "qfp_..." 
  },

  "members": [
    { "claim_ir_ref": "sha256:...", "claim_id": "clm_1" },
    { "claim_ir_ref": "sha256:...", "claim_id": "clm_2" }
  ],

  "outcome": "RESOLVED|UNRESOLVED|REJECTED",

  "selected_resolved_claim_id": "rcl_...",

  "reason_codes": ["INSUFFICIENT_EVIDENCE|POLICY_BLOCK|SOURCE_CONFLICT|AMBIGUOUS_INPUT"],
  "rationale": "short structured summary"
}
```

Rules (normative):

* If `outcome="UNRESOLVED"`, `selected_resolved_claim_id` MUST be absent (or null) unless the selection is explicitly marked as a non-binding proposal (proposals MUST NOT be forwarded to compilation as truth).
* Conflict groups MUST remain traceable to all original member claims and evidence refs.

---

## 7) Ambiguity groups (render-facing ambiguity)

Ambiguity is first-class. When ambiguity cannot be resolved, it MUST be preserved explicitly so downstream systems can:

* render ambiguity clearly,
* avoid inventing a single answer,
* satisfy validation rules that require ambiguity marking.

```json
{
  "ambiguity_group_id": "ag_...",
  "topic": "slot|entity|relation|time|identity",
  "members": ["rcl_...", "rcl_..."],
  "rendering_hint": "show_options|show_range|refuse_if_required",
  "notes": "why unresolved"
}
```

Rules (normative):

* Every `UNRESOLVED` outcome MUST be representable as at least one ambiguity group (directly or via a conflict group mapping).
* `rendering_hint` MUST be policy-driven and deterministic.

---

## 8) Validation expectations (what Orgo will gate)

Resolved Claim-IR MUST be structured so Validation can enforce:

* schema conformity
* required lineage/provenance fields present
* policy compliance (forbidden classes/operations rejected)
* no-new-facts discipline (all claims trace to lineage and evidence)
* ambiguity marking rules (unresolved conflicts must be explicitly tagged/grouped)

If Validation fails, compilation MUST NOT proceed.

---

## 9) Serialization and ordering (determinism)

To support reproducibility:

* `claims[]` SHOULD be sorted deterministically (e.g., by `triple.subject.id`, `triple.predicate.id`, `triple.object.id`, qualifier fingerprint, then `resolved_claim_id`)
* `conflict_groups[]` and `ambiguity_groups[]` SHOULD be sorted deterministically by their ids
* `reason_codes` SHOULD be stable enums; free-text `rationale` SHOULD be minimal and structured (to reduce nondeterministic drift)

---

## 10) Security / privacy hooks

If confidentiality classes exist:

* Resolved Claim-IR MAY include redactions/confidentiality tags (prefer via `notes.redactions` or `extensions`)
* Sensitive provenance pointers MUST be handled per policy (opaque refs allowed)
* Resolution rationale MUST NOT leak protected data

---

## 11) Conformance tests (required)

A compliant Resolved Claim-IR producer MUST pass:

* **Lineage completeness**: each resolved claim references source Claim-IR refs + claim ids + evidence refs
* **Ambiguity correctness**: unresolved conflicts are tagged/grouped; no silent selection
* **No-new-facts**: no claim without lineage/evidence pointers
* **Deterministic ordering**: stable output ordering given identical inputs
* **Policy compliance**: forbidden classes blocked or marked rejected with deterministic reason codes
* **Schema compliance**: required fields present; enums valid

---

## 12) Minimal example (unresolved conflict)

Two sources disagree; ambiguity is preserved:

* conflict group outcome: `UNRESOLVED`
* multiple candidate resolved claims remain traceable
* ambiguity group instructs renderers to show options (or refuse if strict)

This is correct behavior for ambiguous inputs.

