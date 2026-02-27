# Testing & Conformance
Path: `docs/07-implementation-guides/03-testing-conformance.md`

## Status
Normative (minimum conformance suite). Informative sections are labeled.

## Purpose
Define a test suite that proves an implementation conforms to the ecosystem’s contracts:
- **global invariants**
- **typed artifacts + schemas**
- **deterministic gates**
- **no new facts downstream**
- **fail-closed distribution**
- **auditability + replay**

Conformance is proven by automated tests that are reproducible, deterministic, and runnable in CI.

---

## 1) Conformance levels

### Level 0 — Contract presence
Proves interfaces exist and return schema-valid artifacts.

### Level 1 — Gate correctness
Proves stage ordering, hard gates (“no compile on fail”), and fail-closed behaviors.

### Level 2 — Determinism + replay
Proves deterministic outputs under frozen inputs/config, including golden-file stability.

### Level 3 — Audit-grade traceability
Proves end-to-end traceability from input snapshots to outputs (Exchange/Pack/Render Bundle), including `trace_map` coverage rules.

An implementation SHOULD target Level 2 minimum; Level 3 is recommended for production.

---

## 2) Test harness requirements (mandatory)

### 2.1 Fixtures (mandatory)
The conformance suite MUST include a standardized fixture set:

- `fixtures/inputs/`
  - immutable source snapshots (small, medium)
  - provenance metadata (stable)
- `fixtures/claim_ir/`
  - valid Claim-IR batches
  - invalid Claim-IR batches (schema violations)
  - edge cases: ambiguity, conflicting claims, missing evidence
- `fixtures/resolved_claim_ir/`
  - resolved single, resolved multi, unresolved, error cases
- `fixtures/validation_reports/`
  - pass report, fail report with deterministic codes
- `fixtures/exchange/`
  - canonical Exchange payload fixtures (small)
  - expected Exchange hash for canonical serialization
- `fixtures/runtime_packs/`
  - minimal pack payload fixtures + manifest
  - corrupted payload fixtures (tamper tests)
- `fixtures/render/`
  - templates + parameters
  - expected Render Bundles (goldens) including `trace_map`
- `fixtures/orgo/`
  - Case/Task payloads and state transition sequences
  - stage timeline sequences for build records
- `fixtures/ekoh/`
  - ledger events + expected score deltas
  - privacy settings and expected redactions

### 2.2 Canonicalization rules (mandatory)
All hashing and “same input → same output” checks MUST use the declared canonicalization profile (e.g., JSON canonicalization profile) and MUST run with:
- stable key ordering (if applicable)
- stable whitespace rules (if applicable)
- stable timestamp handling (timestamps excluded from content hashing or isolated)

### 2.3 Deterministic seeds (mandatory)
Any probabilistic component MUST support a deterministic mode for tests:
- fixed seed
- fixed model version
- fixed dependency versions
- frozen candidate sources (where applicable)

If deterministic mode is not available, the test suite MUST instead freeze upstream artifacts (e.g., use stored Claim-IR rather than re-extracting).

---

## 3) Global invariant tests (mandatory)

### INV-01 Truth boundary
**Requirement:** canonical truth exists only after validation + compilation.
- Attempt to publish/distribute or render using unvalidated artifacts MUST fail deterministically.
- Attempt to write to Exchange outside the compile stage MUST be rejected.

### INV-02 Typed interfaces
**Requirement:** all cross-node exchanges are schema-valid.
- Every produced artifact MUST validate against its declared schema version.
- Unknown schema versions MUST be rejected.

### INV-03 No compile on fail
**Requirement:** validation fail blocks compilation.
- Given a failing `validation_report`, Orgo MUST not proceed to Exchange/Pack compilation.

### INV-04 No new facts downstream
**Requirement:** rendering may not invent.
- Render outputs MUST contain only statements traceable to validated inputs (via `trace_map`) or must mark uncertainty / refuse.

### INV-05 Fail-closed distribution
**Requirement:** tampered packs never activate.
- If any manifest/payload hash mismatch occurs, activation MUST fail and rollback MUST preserve last-known-good.

### INV-06 Governed feedback
**Requirement:** feedback creates new work; does not mutate canon directly.
- Feedback actions MUST create Orgo Cases/Tasks (or equivalent work items) and MUST NOT modify Exchange in place.

### INV-07 End-to-end traceability
**Requirement:** every assertion is traceable.
- For any factual output claim, there MUST exist a trace path to:
  input snapshot → Claim-IR → Resolved Claim-IR → Validation report → Exchange → Render bundle.

---

## 4) Artifact schema conformance (mandatory)

### ART-01 Schema validation
For each artifact type:
- Provide `valid` fixtures that MUST validate.
- Provide `invalid` fixtures that MUST fail with deterministic error locations and codes.

Artifacts (minimum set):
- Claim-IR
- Resolved Claim-IR
- Validation report
- Exchange manifest
- Runtime Pack manifest
- Render Bundle
- Orgo Case
- Orgo Task
- EkoH ledger event (if implemented as an artifact)

### ART-02 Version compatibility
- Reader MUST reject unknown major versions.
- Reader MAY accept unknown minor versions only if explicitly allowed by policy, and MUST ignore unknown fields safely when allowed.

### ART-03 Content-addressable integrity
- If an artifact declares `hash` and `hash_algorithm`, recomputation MUST match.

---

## 5) Orgo build pipeline conformance (mandatory)

### ORGO-01 Stage order
- Build MUST follow: Ingest → Extract → Resolve → Validate → Compile Exchange → Compile Pack → Publish.
- Any attempt to skip stages MUST be rejected.

### ORGO-02 Gate enforcement
- Validation fail MUST stop the workflow.
- Orgo MUST surface the validation report reference as the failure artifact.

### ORGO-03 Build record completeness
For every build:
- Build record MUST include:
  - input snapshot refs
  - config/blueprint refs
  - policy selections
  - stage timeline (started/ended/status)
  - outputs (refs/hashes)
- Missing required fields MUST fail conformance.

### ORGO-04 Replay safety
- Replay MUST not bypass integrity checks.
- Replay MUST produce identical outputs when upstream artifacts are frozen.

---

## 6) SenTient conformance (mandatory)

### SENT-01 Output structure determinism
- For a fixed Claim-IR fixture, output MUST be schema-valid with stable structure.
- Candidate sorting MUST follow declared deterministic tie-breakers.

### SENT-02 Ambiguity preservation
- When evidence is insufficient, output MUST be `RESOLVED_MULTI` or `UNRESOLVED` (never force a single binding silently).

### SENT-03 No invention
- SenTient MUST not introduce claims not present in Claim-IR.

### SENT-04 Partial failure behavior
- Under induced timeouts/outages, SenTient MUST return a valid batch marking affected claims as `UNRESOLVED` or `ERROR` with stable codes.

---

## 7) Kristal compilation conformance (mandatory)

### KRIS-01 Compile requires validation pass
- Given a fail validation report, compilation MUST abort.

### KRIS-02 Exchange immutability
- Exchange content is immutable once compiled.
- Any attempt to mutate must result in a new Exchange build, not in-place changes.

### KRIS-03 Manifest completeness
- Exchange manifest MUST include:
  - exchange_id/hash
  - blueprint hash
  - policy identifiers
  - pipeline refs (Claim-IR/Resolved/Validation)
  - signatures (if policy requires)

### KRIS-04 Deterministic hashing
- Exchange hash MUST match canonical serialization of the Exchange payload.
- Manifest hash/signature verification MUST pass (where implemented).

### KRIS-05 Runtime Pack derivation
- Runtime Pack MUST reference Exchange and record its own hashes.
- Pack manifest MUST validate and recomputation MUST match.

---

## 8) Architect conformance (mandatory)

### ARCH-STRAT-01 Strategy respects governance (if implemented)
- Strategy outputs MUST be representable as Orgo work (Cases/Tasks) under allowed labels/categories.
- Strategy must not bypass Orgo stage gates.

### ARCH-REND-01 Deterministic rendering
For fixed inputs:
- output MUST be byte-identical (or equivalently canonical-identical) across runs.
- output MUST include `trace_map`.

### ARCH-REND-02 Trace coverage
- Every factual assertion MUST map to at least one validated support pointer.
- If trace coverage is missing:
  - renderer MUST omit the claim OR
  - explicitly mark it uncertain (only if uncertainty exists upstream) OR
  - fail deterministically (policy-dependent)

### ARCH-REND-03 No new facts
- Introduce a “trap” test where the prompt/template tries to elicit new information.
- Render MUST refuse or remain grounded to validated facts only.

---

## 9) Konnaxion distribution conformance (mandatory)

### KONN-01 Fail-closed verification
- Tamper pack payload ⇒ activation MUST fail.
- Tamper manifest ⇒ activation MUST fail.
- Unknown signer/key ⇒ activation MUST fail (unless policy explicitly allows unsigned).

### KONN-02 Atomic activation
- Activation must be atomic: after failure, system MUST remain on last-known-good pack.
- Rollback must succeed deterministically.

### KONN-03 Compatibility checks
- If pack declares incompatible bounds, activation MUST refuse with a deterministic code.

### KONN-04 Cache correctness (recommended)
- Cached packs must be re-verified when policy requires.
- Cache eviction must not break rollback guarantees.

---

## 10) SwarmCraft execution conformance (mandatory if implemented)

### SWARM-01 Task envelope adherence
- Execution must accept only schema-valid task envelopes.
- Invalid envelopes must be rejected with deterministic codes.

### SWARM-02 Telemetry and correlation IDs
- Every execution must emit:
  - task_id
  - correlation_id
  - status transitions
  - produced artifact refs (if any)

### SWARM-03 No canon mutation
- Execution outputs must become inputs to governed cycles; no direct writes to Exchange.

---

## 11) EkoH ledger conformance (mandatory if implemented)

### EKOH-01 Append-only history
- ScoreHistory cannot be rewritten/deleted through standard interfaces.
- Corrections occur via compensating entries.

### EKOH-02 Validated-only merit
- Unvalidated events MUST NOT change expertise scores.

### EKOH-03 Deterministic weighting
- Given fixed scores + policy/config + input vote, weighted_value MUST be deterministic.

### EKOH-04 Privacy enforcement
- public/pseudonym/anonymous settings must be honored in views/exports.
- Internal audit views must remain accessible only under authorization.

---

## 12) Negative tests (mandatory)

The suite MUST include negative tests for:
- schema violations (unknown fields where forbidden, wrong types, missing required fields)
- invalid state transitions (Orgo Case/Task status machines)
- validation failure path (ensures compile never happens)
- trace_map missing coverage (ensures deterministic refuse/omit/uncertain behavior)
- signature verification failure (ensures fail-closed)
- replay attacks / duplicated events (idempotency checks)

---

## 13) Reporting and CI requirements (mandatory)

### Required outputs
A conformance run MUST produce:
- pass/fail summary by section (INV/ART/ORGO/SENT/KRIS/ARCH/KONN/SWARM/EKOH)
- machine-readable report (JSON) with:
  - test id
  - version of the implementation under test
  - blueprint/policy identifiers used
  - artifact refs/hashes for any produced artifacts
  - deterministic failure codes where applicable

### Golden updates policy
Golden files MUST NOT be updated automatically by CI.
Golden updates require:
- an explicit operator action
- an ADR or change record referencing the invariant/contract change
- a version bump where required

---

## 14) Minimal test matrix (mandatory)

A conformant implementation MUST pass at least:

- All Global Invariants: INV-01..INV-07
- All Artifact Schema tests: ART-01..ART-03
- Orgo: ORGO-01..ORGO-04
- SenTient: SENT-01..SENT-04 (if implemented)
- Kristal: KRIS-01..KRIS-05 (if implemented)
- Architect-Render: ARCH-REND-01..ARCH-REND-03 (if implemented)
- Konnaxion: KONN-01..KONN-03 (if implemented)
- SwarmCraft: SWARM-01..SWARM-03 (if implemented)
- EkoH: EKOH-01..EKOH-04 (if implemented)

Implementations that omit a component MUST declare that omission explicitly and MUST still pass all tests applicable to the components they provide.

---

## Appendix A (informative): Recommended directory layout for tests

```text
conformance/
  fixtures/
  goldens/
  suites/
    invariants.spec.ts|py
    artifacts.spec.ts|py
    orgo.spec.ts|py
    sentient.spec.ts|py
    kristal.spec.ts|py
    architect_render.spec.ts|py
    konnaxion.spec.ts|py
    swarmcraft.spec.ts|py
    ekoh.spec.ts|py
  reports/
    last-run.json
````

## Appendix B (informative): Determinism checklist

* frozen inputs and snapshots
* fixed blueprint hash
* fixed policy id
* fixed component versions
* fixed seeds (if any)
* canonical serialization profile locked
* timestamps excluded from content hashing (or isolated to metadata only)

