# Principles & invariants

This page is the “non-negotiables” view of the system. If any of these are violated, treat it as a bug or an incident, not an acceptable tradeoff.

## 1) Truth boundary

- **Canonical truth exists only after compilation** into the canonical truth artifact (the “truth pivot”).
- **Upstream outputs are proposals**, not truth (inputs, extracted claims, resolved claims, validation outputs).
- **Validation is a hard gate**: if validation fails, canonical outputs must not be produced or distributed.

## 2) No redundancy of external contracts

- **Do not copy or restate external artifact contracts** (schemas, canonicalization rules, signing formats, etc.).
- The system points to the pinned external spec and enforces it at boundaries.

## 3) Typed boundaries (no hidden state)

- Components exchange **typed artifacts**, not implicit shared state.
- “Truth” cannot move through side channels; it must go through the declared pipeline and gates.

## 4) Determinism over availability

- Same **pinned inputs + pinned configuration + pinned policies** must yield **identical (or canonically identical)** results.
- Determinism also applies to failure behavior: same pinned context → same class of failure + stable reasons.

## 5) Fail-closed integrity and activation

- If integrity is declared (hashes/signatures/compat rules), **verification is mandatory**.
- If verification cannot be completed or fails, **activation must not proceed**.
- Activation must be **atomic** (no partial activation).

## 6) Immutable canon and governed change

- Canonical artifacts are **immutable once published**.
- Any canon-affecting change is a **new build**, producing new artifacts/IDs (no hot-edits).

## 7) No new facts downstream

- Rendering and execution layers must **not introduce new factual claims**.
- They can only:
  - trace outputs to canonical sources,
  - omit/refuse deterministically under policy,
  - emit governed feedback that becomes new work.

## 8) Offline correctness

- Correctness must not depend on live network calls at the moment of use.
- Runtime behavior is based on verified, activated packs that can be used offline.

## 9) Auditability as a first-class requirement

- Every build/release must leave an auditable trail that can answer:
  - what inputs were used,
  - which policies/configuration applied,
  - which gates ran and why they passed/failed,
  - which outputs were produced and promoted.

## 10) Tenant isolation and pinned trust roots

- Trust roots and acceptance policy are **tenant-scoped** and **pinned**.
- The system must resist downgrade/substitution and cross-tenant trust confusion.

## Quick “invariant check” list

If you’re reviewing a change, confirm:

- Validation failure cannot produce publishable/activatable outputs.
- Any declared integrity mismatch blocks publish/activation (no bypass).
- A component cannot smuggle facts around validation/compile.
- Output is deterministic under pinned context (including error behavior).
- Rendering cannot invent facts and always provides trace coverage or deterministic refusal.
- Activation is atomic and rollback target selection is deterministic.