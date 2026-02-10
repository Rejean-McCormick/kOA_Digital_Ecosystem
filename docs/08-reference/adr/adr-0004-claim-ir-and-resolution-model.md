# ADR-0004: Claim-IR and Resolution Model

- **Status**: Accepted
- **Date**: 2026-02-09
- **Owner**: SenTient / Orgo / Kristal Working Spec
- **Related**:
  - `docs/04-artifacts-contracts/02-claim-ir.md`
  - `docs/04-artifacts-contracts/03-resolved-claim-ir.md`
  - `docs/04-artifacts-contracts/04-validation-report.md`
  - `docs/03-nodes/06-tiferet-sentient.md`
  - `docs/01-principles/01-invariants.md`

---

## Context

The ecosystem requires a strict separation between:
- **extraction** (turning inputs into structured proposals),
- **resolution** (handling conflicts and ambiguity without inventing truth),
- **validation** (deterministic gating),
- and **compilation** (turning validated results into canon).

Without a contract-grade intermediate representation, downstream components (Orgo validation, Kristal compilation, Architect rendering, SwarmCraft execution) cannot be made deterministic, auditable, or “tight.”

A key failure mode to avoid is **implicit resolution** (silently picking a side) and **truth leakage** (treating extracted text as canonical fact).

---

## Decision

### 1) Adopt a two-stage IR model
We standardize two typed artifacts:

1. **Claim-IR**  
   Output of extractors. Contains schema-constrained claims with provenance/evidence pointers and uncertainty flags. Claim-IR is **proposal-level**, not truth.

2. **Resolved Claim-IR**  
   Output of SenTient. Contains normalized claims with explicit resolution status and explicit ambiguity representation. Resolved Claim-IR is still not canon, but is the required input to deterministic validation and compilation.

---

### 2) Make ambiguity first-class (never silently collapse)
Ambiguity and conflict are not treated as errors by default. They are modeled explicitly:

- **Conflict Groups**: competing claims about the same slot (e.g., subject+predicate) are grouped.
- **Ambiguity Groups**: unresolved conflicts are carried forward as explicit ambiguity objects with rendering hints.

Resolution MUST explicitly yield one of:
- `RESOLVED` (a preferred assertion selected/merged with justification)
- `UNRESOLVED` (no justified selection; ambiguity preserved)
- `REJECTED` (policy block or insufficient evidence)
- `MERGED` (equivalent claims consolidated; still may be RESOLVED/UNRESOLVED at group level)

This prevents “metaphor drift” and prevents downstream systems from manufacturing certainty.

---

### 3) Enforce “no new facts” at the IR boundary
- Claim-IR producers MUST NOT claim canon.
- SenTient MUST NOT introduce new facts. It may only:
  - normalize,
  - merge equivalent claims,
  - select among competing claims if justified by evidence/policy,
  - or preserve ambiguity.

Any resolution decision must be explainable via:
- source Claim-IR lineage,
- evidence pointers,
- and applied resolution rules.

---

### 4) Require lineage and provenance completeness
For every claim in both IR layers:
- lineage MUST include upstream IDs (Claim-IR IDs, claim IDs),
- provenance MUST include input snapshot pointers (direct or opaque),
- evidence references MUST be supported by provenance pointers (subject to policy).

This is required to support:
- deterministic validation,
- compilation provenance integrity,
- and Architect-Render trace maps.

---

### 5) Make IR suitable for deterministic gates
The IR documents must support deterministic processing by:
- version-pinning (mandate/blueprint/policies),
- stable serialization and deterministic ordering rules,
- engine/config identification (version + config_hash),
- explicit reason codes for rejects/deferrals.

---

## Consequences

### Positive
- Downstream components can be specified as deterministic consumers of typed inputs.
- Ambiguity becomes safe: renderers can show “unknown/contested” instead of hallucinating.
- Validation can be implemented as reproducible gates with reason codes.
- Compilation can be reproducible and content-addressed (when enabled).
- Traceability becomes end-to-end: source → Claim-IR → Resolved Claim-IR → Validation → Canon → Render trace_map.

### Trade-offs
- Requires disciplined schema design and strict payload hygiene.
- Adds upfront complexity (conflict groups, ambiguity groups, lineage).
- Some “simple” questions will produce explicit ambiguity rather than a single answer unless justified.

---

## Alternatives considered

### A) Single IR artifact (extractors output ready-for-validation)
Rejected. This collapses extraction and resolution, encourages silent conflict handling, and makes it harder to enforce “no new facts” and ambiguity preservation.

### B) Free-text resolution notes without structured conflict/ambiguity objects
Rejected. Free-text rationale is not reliably machine-checkable and cannot support deterministic validation or trace guarantees.

### C) Always force resolution (must pick a winner)
Rejected. This violates the ecosystem’s ambiguity discipline and would push invention downstream (especially into rendering).

---

## Implementation notes

- Claim-IR and Resolved Claim-IR MUST have schemas in `docs/04-artifacts-contracts/schemas/`.
- Extractors MUST output Claim-IR only, never “truth.”
- SenTient MUST output Resolved Claim-IR with explicit statuses and grouping.
- Orgo validation gates MUST treat Resolved Claim-IR as the required input.
- Kristal compilation MUST only consume validated Resolved Claim-IR and MUST refuse compilation on validation fail.
- Architect-Render MUST ground factual output using lineage from canon back to Resolved Claim-IR and provenance pointers.

---

## References

- `docs/04-artifacts-contracts/02-claim-ir.md`
- `docs/04-artifacts-contracts/03-resolved-claim-ir.md`
- `docs/01-principles/01-invariants.md`
- `docs/05-protocols-flows/01-conscience-input-path.md`
- `docs/05-protocols-flows/04-feedback-path.md`
