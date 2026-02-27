# ADR-0002: Determinism Policy (kOA)

**Status:** Accepted  
**Date:** 2026-02-27  
**Decision Owner:** kOA Ecosystem Architecture  
**Scope:** kOA determinism expectations for pipeline gates, distribution, rendering, and operations.  
**Depends on:** Kristal v4 (pinned). Normative artifact formats are defined in Kristal; this ADR defines kOA policy and enforcement.

---

## 1) Context

kOA is a governed ecosystem where **truth** is produced only at the validation→compile boundary, and distribution/activation must be safe under offline and adversarial conditions. Determinism is required to make:
- rebuilds and comparisons meaningful,
- integrity verification stable across toolchains,
- rollouts safe (repeatable activation/rollback),
- audits reproducible.

Kristal v4 defines canonicalization/hashing/signature semantics for Kristal artifacts. kOA must adopt a clear policy for:
- what must be deterministic,
- what may vary (and how variance is recorded),
- what gates enforce determinism,
- how compatibility is handled across versions.

---

## 2) Decision

### 2.1 Determinism levels
kOA defines three determinism levels for components/stages:

1. **Strict Determinism**
   - Same pinned inputs + blueprint + policy selections + toolchain versions → identical outputs (byte-identical or canonical-identical).
   - Required for: validation gate decisions, Exchange/Runtime Pack compilation targets, verification inputs.

2. **Operational Determinism**
   - Outputs may include benign operational variance (timestamps, attempt IDs), but the *deterministic core* must be stable and separately hashed/signed.
   - Allowed for: Build Records, Release Records, telemetry envelopes.

3. **Presentation Determinism**
   - Rendering must be deterministic for the same pinned Exchange + templates + params, but may include runtime-dependent formatting constraints (screen size) if recorded and traceable.
   - Required for: Render bundles, trace maps, refusal/omission events.

### 2.2 Required enforcement points (gates)
kOA enforces determinism at these gates:

- **Validation gate (hard):** deterministic pass/fail and stable error codes for the same resolved inputs.
- **Compile gate:** compile must not run unless validation gate is PASS (“no compile on fail”).
- **Verification gate (distribution/activation):** fail-closed verification of signatures/hashes/compat policy.
- **Render gate:** deterministic outputs + trace coverage; no new facts.
- **Rollback gate:** deterministic rollback decision for a given trigger and stored history.

### 2.3 What must be pinned
To claim determinism at any level, these must be pinned and recorded:

- Input snapshot refs (content-addressed)
- Blueprint ref (schemas/config/templates)
- Mandate ref (policy context), if applicable
- Policy selections (enabled profiles)
- Toolchain identities (component name + version + build hash if available)

---

## 3) Consequences

### 3.1 Allowed variance (must be recorded)
The following may vary without violating kOA determinism policy, provided it is recorded:

- attempt/retry identifiers
- processing timestamps in operational records
- machine/environment metadata (hostnames, regions)
- telemetry sampling (if explicitly configured and recorded)

### 3.2 Disallowed variance
The following is disallowed:

- changing gate outcomes under the same pinned context
- producing different Exchange/Runtime Pack identities for the same pinned context (except when explicitly versioned and recorded as a change)
- accepting unverifiable artifacts (“best effort” verification)
- rendering new facts not traceable to validated lineage

---

## 4) Implementation requirements

### 4.1 Recording requirements (kOA artifacts)
- Build Record must record pinned context + gate outcomes + artifact refs.
- Release Record must record promotion intent + target channels/cohorts + activation expectations.

### 4.2 Test requirements
A conformance suite must include:
- deterministic rebuild tests (same pinned context → same canonical identities)
- verification fail-closed tests (bad signature/hash → no activation)
- rollback determinism tests (same trigger history → same rollback outcome)
- render determinism + trace coverage tests

---

## 5) Migration and compatibility

- Determinism requirements apply from the adoption of the pinned Kristal v4 dependency onward.
- Any relaxation or tightening requires a new ADR and a documented rollout window.
- Legacy acceptance is controlled by `docs/40-integration/kristal-v4/legacy-compat.md`.

---

## 6) References (non-normative)

- Kristal v4 pinned dependency: `docs/40-integration/kristal-v4/pinned-dependency.md`
- kOA pipeline runbook: `docs/50-operations/pipeline.md`
- kOA conformance: `docs/40-integration/kristal-v4/conformance.md`