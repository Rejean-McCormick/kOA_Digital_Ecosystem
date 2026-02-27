# ADR-0003: Konnaxion Activation and Rollback

- **Status**: Accepted
- **Date**: 2026-02-09
- **Owner**: Konnaxion (Distribution) + Orgo (Release Control Plane)
- **Scope**: Runtime Pack verification, activation gating, downgrade prevention, rollback, offline/storage behavior
- **Out of scope**: Kristal artifact formats/schemas (normative in Kristal v4); Konnaxion-Social behaviors

---

## Context

Konnaxion distributes Runtime Packs to edge nodes/devices and activates them locally. A bad pack (corrupted, tampered, incompatible, substituted, revoked, or downgraded) can break correctness, safety, reproducibility, and user trust.

We need a deterministic, fail-closed, auditable activation and rollback contract that:
- prevents partial activation,
- prevents downgrade vulnerabilities,
- supports fast rollback,
- preserves offline correctness.

Kristal v4 defines the Runtime Pack and its manifest schema; kOA defines activation policy and operations.

---

## Decision

### 1) Verification-before-activation (fail-closed)

Konnaxion MUST verify a candidate pack before activation.

Verification MUST include (deterministic, profile-defined):
1) Manifest parsing + schema validation (Kristal v4 schema)
2) Integrity verification (as declared)
3) Signature verification (as declared) using pinned trust roots
4) Required-file presence checks for all manifest-declared required paths

If any verification step fails:
- activation MUST NOT proceed
- Konnaxion MUST remain on the current active pack (or last-known-good)
- deterministic diagnostics MUST be emitted (stable reason codes)

### 2) Compatibility checks before activation (mandatory)

Before activation, Konnaxion MUST validate compatibility between:
- pack-declared contract version(s) / required profiles
- local runtime supported versions / capabilities

On incompatibility:
- do not activate
- emit deterministic diagnostics
- remain on current active pack

### 3) Atomic activation (mandatory)

Activation MUST be an atomic switch:
- either the new pack becomes the active pointer, or
- no state changes occur.

Partial activation is forbidden.

### 4) Downgrade prevention with persisted safety state (mandatory)

Konnaxion MUST implement downgrade prevention using deterministic, persisted per-channel state.

Minimum required rule:
- MUST NOT activate a release/version lower than the highest previously activated release in the same channel, unless an explicit policy-authorized rollback is present.

Required persisted state (per channel):
- `highest_activated_release_id`
- (recommended) `highest_seen_release_id`
- last successful verification metadata (artifact_id, signer, timestamp)

Revocation-aware rule:
- MUST NOT activate packs listed as revoked by a verified channel control surface.

Substitution safety:
- If a pack is presented with the same release identifier but different immutable artifact identifier, treat as a hard error unless an explicit reissue authorization is present in a verified control surface.

### 5) Deterministic rollback (mandatory)

Konnaxion MUST support at least one rollback mode:
- **Pinned rollback**: activate an explicitly pinned known-good pack
- **Last-known-good rollback**: activate the most recent previously active verified pack still present and verified

Rollback MUST be:
- explicit (never silent)
- policy-authorized (see below)
- deterministic given the same trigger sequence and verified inputs

Rollback authorization MAY be conveyed via:
- a verified channel index that includes an explicit rollback authorization record, and/or
- an explicit operator action that is auditable and policy-gated (deployment-specific)

Rollback triggers MAY include:
- explicit operator action (recommended)
- verified index updates (e.g., “current revoked”, rollback authorization added)
- local runtime health signals (optional; must be policy-defined)

### 6) Trust roots pinned; offline correctness (mandatory)

Trust roots MUST be pinned per channel scope (tenant/environment/region/audience as applicable) and MUST be verifiable offline.

Trust roots MUST NOT be fetched over the network at activation time as a dependency for correctness.

### 7) Signed channel index as recommended control surface

Konnaxion SHOULD consume a signed per-channel distribution index (e.g., `pack_index.json`).

If present, it MUST be verified using pinned trust roots (fail-closed) and treated as the primary control surface for:
- what is current/latest,
- what is pinned,
- minimum allowed release,
- revocations,
- (optionally) rollback authorization.

### 8) Offline caching and storage contract (mandatory to define)

Konnaxion MUST define deterministic storage + cache policies that support:
- multiple installed versions per channel
- atomic activation pointers
- garbage collection with pinning rules
- retaining at least: active + last-known-good (+ pinned set if used)

If offline, Konnaxion MUST:
- continue serving from the active pack
- not attempt activation requiring network-dependent trust roots
- optionally surface “stale pack” metadata

Recommended layout (example; not normative):
- `/<tenant>/<env>/<channel>/packs/<runtime_pack_id>/...`
- `/<tenant>/<env>/<channel>/active -> <runtime_pack_id>`
- `/<tenant>/<env>/<channel>/state.json` (persisted safety metadata)

---

## Consequences

### Positive
- Prevents silent regressions and downgrade vulnerabilities
- Makes activation decisions auditable and reproducible
- Enables safe offline-first operation and deterministic recovery

### Costs / Requirements
- Requires persistent per-channel state and stable diagnostic codes
- Requires clear policy surfaces for rollback authorization and health-triggered rollback
- Requires deterministic cache/eviction behavior and pinned retention rules

---

## Conformance tests (required)

A conformant Konnaxion-Distribution implementation MUST provide tests for:
- schema validation success/failure (fail-closed)
- integrity verification success/failure (fail-closed)
- signature verification success/failure (fail-closed)
- atomic activation (no partial state)
- downgrade prevention behavior (monotonic + persisted state)
- substitution safety behavior
- revocation-aware activation blocking
- rollback modes and rollback determinism
- offline behavior (serve active; no network dependency for trust roots)
- cache eviction respects active/last-known-good/pinned retention rules

---

## References (kOA)

- `docs/20-nodes/chesed-konnaxion.md`
- `docs/50-operations/releases.md`
- `docs/50-operations/rollback.md`
- `docs/40-integration/kristal-v4/contract-pointers.md`
- `docs/40-integration/kristal-v4/koa-profile.md`