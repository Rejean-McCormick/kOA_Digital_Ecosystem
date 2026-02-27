# Testing (kOA)

**File:** `docs/60-guides/testing.md`  
**Normative scope:** kOA testing expectations and test organization.  
**Non-normative:** Kristal artifact schemas/formats (external); this guide references the pinned Kristal v4 dependency for schema validation.

---

## 1) Purpose

Define the kOA test strategy and minimum suite required to ensure:
- deterministic behavior across the pipeline,
- correct stage gating (“no compile on fail”),
- fail-closed verification and activation,
- conformance to the pinned Kristal v4 schemas where kOA produces/consumes Kristal artifacts,
- reliable operations (rollback, incident handling, observability).

---

## 2) Test layers

### 2.1 Unit tests (component-local)
Scope:
- deterministic pure functions
- schema helpers
- validators
- policy evaluators
- hash/signature utilities (kOA side)

Examples:
- normalization helpers (SenTient)
- manifest parsing and strict validation logic
- downgrade-prevention logic (Konnaxion)

### 2.2 Contract tests (boundary artifacts)
Scope:
- producer emits artifact that validates against its schema
- consumer rejects malformed artifacts
- stable error codes and failure shapes

Rule:
- For Kristal artifacts, use the **pinned Kristal v4 schemas** (do not copy schemas into kOA).  
See: `docs/40-integration/kristal-v4/pinned-dependency.md`.

### 2.3 Integration tests (multi-component)
Scope:
- Orgo → Extract → SenTient → Validate → (Kristal compile) → Konnaxion verify/activate
- “happy path” and controlled failures
- telemetry emission

### 2.4 End-to-end tests (release simulation)
Scope:
- build + publish + staged rollout
- cohort targeting
- rollback under controlled conditions
- audit evidence completeness

---

## 3) Minimum required test suite (kOA)

### 3.1 Determinism
- **Rerun determinism**: identical pinned inputs/resources/policies → identical outputs
- **Ordering determinism**: stable ordering for lists/maps; no nondeterministic iteration
- **Golden fixtures**: fixed fixtures for:
  - Claim-IR
  - Resolved Claim-IR
  - Validation Report
  - Runtime Pack verification flows

### 3.2 Gate semantics
- **No compile on fail**: validation failure prevents any publish/packaging step
- **Explicit failure artifacts**: failures produce structured reports, not partial truth

### 3.3 Schema conformance
- Validate all emitted artifacts against their schemas.
- For Kristal artifacts, validate against pinned Kristal v4 schemas.
- For kOA-native artifacts (Orgo Task/Case, Build Record, Release Record, etc.), validate against kOA schemas under `docs/30-artifacts/schemas/`.

### 3.4 Verification + activation (fail-closed)
- Corrupted payload hash → activation must fail
- Missing payload → activation must fail
- Invalid signature/trust root → activation must fail
- Schema invalid → activation must fail
- Compatibility mismatch / downgrade attempt → activation must fail

### 3.5 Rollback
- Rollback to last-known-good is deterministic
- Rollback emits required audit events and telemetry
- Partial activation cannot occur (atomic switch verified)

### 3.6 Observability and audit
- Required structured events are emitted for:
  - stage start/finish
  - validation outcomes
  - verification outcomes
  - activation/rollback
- Evidence links exist for:
  - Build Record
  - Release Record
  - active pack reference (and history)

---

## 4) Test data and fixtures

### 4.1 Fixture categories
- **tiny**: minimal valid artifacts for fast checks
- **golden**: representative “real” artifacts for regression
- **adversarial**: malformed/edge-case artifacts
- **compat**: multi-version artifacts to test downgrade prevention and compatibility policies

### 4.2 Fixture management
- Fixtures must be content-addressed and versioned
- Any fixture change requires:
  - reason
  - expected behavior update
  - snapshot of previous expected outputs (for auditability)

---

## 5) Suggested CI gates

Minimum CI pipeline stages:
1. Lint / static checks
2. Unit tests
3. Contract tests (schemas)
4. Integration tests (pipeline spine)
5. End-to-end rollout simulation (nightly or pre-release)
6. Conformance report publication (kOA-native artifact, optional)

---

## 6) Failure triage playbook (testing perspective)

When a test fails, capture:
- pinned versions (code, policy bundle, resource bundles, Kristal dependency)
- input references (snapshots/fixtures)
- full diagnostics artifacts (validation report, verification logs)
- reproduction command or harness

---

## 7) Related docs

- Conformance checklist: `docs/40-integration/kristal-v4/conformance.md`
- Operations pipeline: `docs/50-operations/pipeline.md`
- Rollout/rollback: `docs/50-operations/rollback.md`
- Determinism: `docs/10-system/determinism.md`