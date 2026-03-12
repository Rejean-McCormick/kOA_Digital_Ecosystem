# Kristal v4 Conformance (kOA)

**File:** `docs/40-integration/kristal-v4/conformance.md`  
**Normative scope:** kOA requirements for producing/consuming Kristal artifacts (conformance gates).  
**External normative source:** Kristal v4 (pinned) schemas and spec.

---

## 1) Purpose

Define the kOA conformance requirements for any component that:
- produces Kristal-adjacent artifacts for compilation, or
- consumes Kristal outputs (Exchange / Runtime Packs), or
- performs verification/activation (Konnaxion distribution facet).

This document is a **kOA gate checklist**. It does not restate Kristal artifact schemas; it points to the pinned Kristal v4 dependency.

---

## 2) What “conformant” means in kOA

A kOA implementation is conformant if it satisfies all of the following:

1. **Stage gating is enforced**: Orgo ensures stage order and hard gates (no compile on fail).
2. **Typed boundaries**: the system exchanges typed artifacts at boundaries; no implicit truth.
3. **Kristal artifacts validate** against the pinned Kristal v4 schemas before:
   - publication (Exchange),
   - activation (Runtime Pack).
4. **Fail-closed verification**: any mismatch in integrity, schema, or compatibility prevents activation.
5. **Determinism**: reruns with pinned inputs/resources/policies yield identical (or canonical-identical) outputs.

---

## 3) Pinned Kristal v4 dependency

kOA MUST pin:
- Kristal version (tag/commit)
- schema paths used for validation (Exchange Manifest, Runtime Pack Manifest, Validation Report, etc.)
- canonicalization profile/version expectations (as specified by Kristal v4)

See: `docs/40-integration/kristal-v4/pinned-dependency.md`.

---

## 4) Producer conformance checks (upstream of Kristal compile)

### 4.1 Claim-IR producers (extractors)
MUST:
- emit schema-valid Claim-IR for the pinned Kristal v4 contract
- include stable evidence pointers and deterministic ordering
- avoid generating “truth claims” that bypass validation (no side channels)

SHOULD:
- emit stable diagnostic codes for extraction errors

### 4.2 SenTient (resolution) outputs
MUST:
- emit schema-valid Resolved Claim-IR (pinned Kristal v4 contract)
- preserve ambiguity explicitly
- ensure deterministic ordering and stable scoring outputs

---

## 5) Validator conformance (pre-compile gate)

### 5.1 Validation report artifact
MUST:
- be produced deterministically from pinned inputs
- include enough detail for operators to reproduce failures
- include stable codes for policy/schema violations

### 5.2 Gate rule
MUST:
- prevent compilation/publishing of Exchange/Runtime Pack when validation status is failing (no compile on fail)

---

## 6) Compiler/packaging conformance (Kristal outputs)

### 6.1 Exchange publication
MUST:
- validate the Exchange Manifest and any referenced required artifacts against pinned Kristal v4 schemas
- produce content-addressed identity as defined by Kristal v4
- publish immutably (no in-place mutation)

### 6.2 Runtime Pack production
MUST:
- validate Runtime Pack Manifest against pinned Kristal v4 schema
- bind Runtime Pack identity to the declared Exchange reference(s) per Kristal v4
- include all payloads referenced by the manifest

---

## 7) Consumer conformance (Konnaxion / runtime)

### 7.1 Verify-before-activate (fail-closed)
Konnaxion MUST verify, in order:

1. **Schema validity**
   - Validate the Runtime Pack Manifest against pinned Kristal v4 schema
2. **Integrity**
   - Verify declared hashes for all payloads
   - Verify signatures according to the pinned trust policy
3. **Compatibility**
   - Enforce compatibility rules (version constraints, downgrade prevention, tenant policy)
4. **Atomic activation**
   - Activate as a single atomic switch
   - Ensure deterministic rollback to last-known-good

Any failure MUST stop activation and produce an auditable event.

### 7.2 Rollback
MUST:
- be deterministic given the same triggers
- produce an activation/rollback record and emit telemetry

---

## 8) Determinism tests (minimum suite)

Implementations MUST pass:

- **Rerun determinism**: same pinned inputs/resources/policies → identical outputs
- **Ordering determinism**: stable ordering for lists/maps in produced artifacts
- **Golden tests**: fixed fixtures for Claim-IR/Resolved Claim-IR/Validation Report
- **Compatibility tests**: reject incompatible Runtime Packs; prevent downgrades
- **Failure tests**: corrupt payload hash → fail-closed; invalid schema → fail-closed

---

## 9) Audit and evidence requirements

kOA MUST be able to show, for any active Runtime Pack:

- the originating Build Record reference
- the pinned Kristal version used for schema validation
- verification outcomes (hash/signature/compat)
- the activation timeline (events)
- the rollback history (if any)

---

## 10) Conformance reporting (kOA-native)

kOA SHOULD maintain a kOA-native **conformance report** per release:
- test suite version
- Kristal pinned version
- verification policy version
- pass/fail summary + links to evidence artifacts

(If you formalize this, it belongs under `docs/30-artifacts/` as a kOA artifact.)

---