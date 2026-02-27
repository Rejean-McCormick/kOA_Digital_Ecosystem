# ADR-0002: Truth Boundary and Canonicalization

## Status
Accepted

## Context
The ecosystem ingests uncertain, heterogeneous inputs and produces artifacts consumed by renderers, executors, and distribution layers. Without an explicit truth boundary and canonicalization rules, the system risks:
- “truth drift” (downstream components inventing or mutating facts),
- irreproducible builds across toolchains,
- ambiguous integrity verification,
- loss of auditability from input → decision → output.

We therefore need a single, enforceable point where “truth becomes canonical,” plus byte-stable canonicalization and identity rules.

## Decision
1) **Truth boundary**  
Canonical truth exists **only after** deterministic validation and compilation into a **Kristal Exchange** artifact. Everything upstream is proposal, not truth.

2) **No compile on fail (hard gate)**  
If validation fails, compilation MUST NOT proceed. The pipeline halts with a Validation Report.

3) **Canonical truth is immutable**  
Once compiled, Exchange is immutable. Any change must produce a new version/artifact; no in-place mutation.

4) **Canonicalization is normative**  
All content-addressed identities use a single declared canonicalization profile:
- Canonical JSON is defined by **RFC 8785 JSON Canonicalization Scheme (JCS)**.
- Implementations must not add canonicalization steps that alter byte output beyond the declared profile.

5) **Content-addressed identities are normative**
- `kristal_id` is computed deterministically from the canonicalized Exchange payload (excluding signatures).
- Runtime Packs MUST have a stable `pack_id` derived from their declared reproducibility surface (at minimum manifest + referenced payload hashes).

6) **Integrity verification is fail-closed**
If an artifact declares integrity material (hashes/signatures/key refs), verifiers MUST fail closed on any mismatch, malformed fields, or ambiguity.

7) **Governed change for feedback**
Feedback, corrections, votes, and telemetry signals MUST NOT mutate Exchange directly. They create governed work (Cases/Tasks) that may produce new validated artifacts through a new pipeline run.

## Details

### A) Truth boundary definition
- Pre-truth artifacts (mutable): inputs, Claim-IR, Resolved Claim-IR, Validation Report.
- Canonical truth artifact (immutable): Kristal Exchange (Exchange + manifest).
- Derived distribution artifacts: Runtime Pack + manifest (verifiable derivation from Exchange).

### B) Canonicalization and identity rules
- Exchange canonicalization uses JCS.
- `kristal_id = sha256( JCS( exchange_without_signatures ) )`
- Runtime Pack identity is content-addressed from the pack’s declared reproducibility surface (manifest + payload hashes at minimum).
- Artifacts must declare canonicalization profile + version for cross-toolchain comparison.

### C) Gate semantics
- The Validation Report is the gate output and must exist for every validation attempt.
- Validation failure is terminal for that run: compilation and publish are forbidden.

### D) Downstream constraints
- Renderers may only render facts supported by validated inputs (no new facts).
- Distribution may only activate packs after verification (fail closed) and must support deterministic rollback.

## Consequences
### Positive
- Eliminates “truth leakage” from downstream layers into canon.
- Enables reproducible builds and cross-toolchain comparison.
- Makes integrity verification objective and automatable.
- Preserves a clear audit trail: input snapshots → decisions → Exchange → pack → outputs.

### Negative / Costs
- Increased strictness: more refusals/halts when data is ambiguous or under-specified.
- Requires schema discipline and tooling to generate/validate canonical manifests.
- Requires operational support for replay, rollback, and deterministic build environments.

## Alternatives considered
1) **Allow downstream components to write/patch truth**  
Rejected: breaks immutability/auditability and makes the system non-deterministic.

2) **Canonical truth at “resolution” (SenTient output)**  
Rejected: resolution is still proposal until it passes deterministic validation + compilation.

3) **Best-effort verification (warn but proceed)**  
Rejected: violates fail-closed guarantees and allows tampered artifacts to activate.

4) **Non-content-addressed IDs (UUIDs only)**  
Rejected: prevents cross-toolchain equivalence and reproducibility proofs.

## Follow-ups
- Formalize missing schemas (mandate bundle, plan envelope, task envelope, render bundle schema) so the truth boundary remains enforceable at every interface.
- Implement conformance tests:
  - no compile on fail,
  - Exchange immutability,
  - deterministic `kristal_id`/`pack_id`,
  - fail-closed verification,
  - “no new facts” rendering with trace coverage,
  - feedback creates governed work and cannot mutate Exchange.

## Related
- ADR-0003: Deterministic gates and stage ordering
- ADR-0005: Exchange + Runtime Pack formats
- ADR-0006: Render bundle + trace map requirements
- ADR-0007: Verification, activation, rollback (distribution)
```

Sources used: truth boundary, immutability, no-new-facts, fail-closed, governed feedback ; canonisation/distribution/feedback path constraints  ; JCS canonicalization, `kristal_id` / `pack_id`, fail-closed verification, reproducibility requirements   ; deterministic rendering + no new facts + trace bundle requirements  
