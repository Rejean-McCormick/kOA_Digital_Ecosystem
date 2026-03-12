# Integrator Guide (kOA)

## Purpose
This guide explains how to integrate a system into the kOA Digital Ecosystem as a producer/consumer of kOA-native operational artifacts and as a consumer/producer of Kristal artifacts **via the pinned Kristal v4 dependency**.

This guide does **not** redefine Kristal schemas or artifact formats.

---

## 1) Integration entry points

### A) Integrating as an input producer
You provide upstream inputs that eventually become canonical via validation + Kristal compilation.

Typical integrations:
- ingest raw data feeds into the input pipeline
- submit curated evidence bundles
- provide domain-specific extractors that emit Claim-IR candidates (if you operate that stage)

You must:
- provide stable provenance metadata
- support content-addressed input snapshots (or equivalent reproducible input references)
- use deterministic processing settings where required by policy

### B) Integrating as a pipeline stage service
You implement a stage component (e.g., extractor, resolver, validator adjunct) controlled by Orgo.

You must:
- accept inputs as explicit artifact references (not implicit shared state)
- produce outputs as explicit artifacts (content-addressed where applicable)
- emit stable reason codes on failure
- be idempotent under retry

### C) Integrating as a distribution/activation consumer
You run Konnaxion or consume Konnaxion outputs.

You must:
- verify and activate runtime artifacts fail-closed
- support deterministic rollback
- emit operational state/telemetry back to Orgo

See `docs/50-operations/rollback.md` and `docs/30-artifacts/konnaxion-state.md`.

---

## 2) Normative sources (do not duplicate)

### Kristal artifacts
All Kristal artifacts (Exchange, Runtime Pack, Validation Report, etc.) are externally specified.

Use:
- `docs/40-integration/kristal-v4/pinned-dependency.md`
- `docs/40-integration/kristal-v4/contract-pointers.md`
- `docs/40-integration/kristal-v4/conformance.md`

### kOA-native artifacts
kOA-native operational artifacts are specified here:
- `docs/30-artifacts/` and `docs/30-artifacts/schemas/`

---

## 3) Required integration contracts

### A) Identity and references
Your integration should treat artifact references as immutable and stable:
- never overwrite canonical artifacts in place
- always create a new artifact/ref for changes
- propagate references through Orgo (Build/Release records) rather than “out-of-band” IDs

### B) Determinism requirements
If your component participates in a deterministic gate, you must:
- pin non-deterministic dependencies (models, templates, configs)
- avoid time-based randomness unless explicitly recorded as an input
- produce stable outputs for the same inputs

### C) Fail-closed behavior
If you cannot verify or validate:
- do not proceed
- return a stable failure reason code
- emit enough telemetry for an operator to diagnose without guessing

---

## 4) Integration patterns

### Pattern 1 — Upstream feed → Orgo ingest
1. Your system publishes inputs + provenance.
2. Orgo ingests and records snapshot references.
3. Downstream stages proceed under Orgo control.

Checklist:
- provenance fields complete
- snapshot references immutable
- inputs discoverable for rebuild

### Pattern 2 — External validator adjunct
1. Orgo runs the canonical validation gate.
2. Your service provides additional checks (lint, domain constraints, policy evaluation).
3. Results are returned as structured findings.
4. Orgo decides pass/fail under policy.

Checklist:
- deterministic check outputs
- stable finding IDs and reason codes
- no mutation of truth artifacts

### Pattern 3 — Runtime Pack distribution mirror
1. Your system mirrors pack distribution to edge/offline environments.
2. Konnaxion verifies signatures/hashes.
3. Activation occurs atomically, rollback deterministic.

Checklist:
- fail-closed verification
- multi-version cache supported
- rollback path tested

---

## 5) Required telemetry

You should emit (minimum):
- build/release correlation IDs (from Orgo)
- stage timing (start/end/fail)
- stable error codes + human-readable summaries
- references to logs/traces (not raw blobs in-band)

If you operate Konnaxion:
- emit Konnaxion State records (`docs/30-artifacts/konnaxion-state.md`)
- report activation/rollback outcomes and health signals

---

## 6) Security expectations

Integrators must:
- secure any signing/verifying keys (never embed in jobs or logs)
- authenticate Orgo control-plane requests
- enforce least privilege for stage execution workers
- log privileged actions with audit-grade detail

---

## 7) Compatibility and migration

If you are migrating from earlier Kristal conventions:
- follow `docs/40-integration/kristal-v4/legacy-compat.md`
- do not emit legacy spellings at boundaries
- track normalization metrics until legacy usage is eliminated

---

## 8) Go-live checklist

- [ ] You can run end-to-end with deterministic settings (where required).
- [ ] You handle retries idempotently.
- [ ] You emit stable reason codes on failure.
- [ ] You validate against the correct schemas (kOA-native locally; Kristal via pinned dependency).
- [ ] You verify/activate/rollback fail-closed (distribution consumers).
- [ ] You provide telemetry refs (logs/traces/metrics) tied to build/release IDs.
- [ ] You have an operator escalation path for failed gates/rollouts.