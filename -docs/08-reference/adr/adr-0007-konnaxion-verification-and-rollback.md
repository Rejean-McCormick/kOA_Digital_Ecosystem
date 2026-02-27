# ADR-0007: Konnaxion Verification and Rollback

- **Status**: Accepted
- **Date**: 2026-02-09
- **Owner**: Konnaxion / Kristal Release Spec
- **Related**:
  - `docs/03-nodes/04-chesed-konnaxion.md`
  - `docs/04-artifacts-contracts/06-runtime-pack.md`
  - `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
  - `docs/01-principles/01-invariants.md`
  - `docs/06-operations/02-release-management.md`

---

## Context

Runtime Packs are portable artifacts that must be distributed to clients and activated locally. A corrupted, tampered, incompatible, or downgraded pack can break:
- correctness (“truth” serving wrong projections),
- safety (running unauthorized code/assets),
- reproducibility (cannot replay results),
- user trust (silent regressions).

The ecosystem requires **fail-closed integrity** at the edge and a deterministic recovery strategy.

---

## Decision

### 1) Verification-before-activation (fail-closed)
Konnaxion MUST verify a pack before activation.

Verification MUST include (profile-dependent, but deterministic):
1) manifest parsing + schema validation
2) integrity verification:
   - if `hashes` are present, verify at least `manifest_sha256`
   - if per-file hashes are declared in `files[]`, verify those hashes per policy
3) signature verification:
   - if `signatures[]` present, verify signatures using pinned trust roots
4) required file presence:
   - every required `files[].path` exists

If any verification step fails:
- activation MUST NOT proceed
- the system MUST remain on the current active pack (or last-known-good)
- emit deterministic error codes and reason codes

This is a hard “fail-closed” boundary.

---

### 2) Compatibility checks before activation (mandatory)
Konnaxion MUST validate compatibility between:
- the pack’s `compatibility.query_contract_version`
- the client runtime’s supported contract versions

Konnaxion MUST also check:
- required capabilities present (if declared)
- required projections present for consuming modules
- locale compatibility where relevant

On incompatibility:
- do not activate
- emit deterministic diagnostics
- remain on current active pack

---

### 3) Atomic activation (no partial state)
Activation MUST be atomic:
- either the new pack becomes the active pack pointer,
- or no state changes occur.

No partial activation is allowed (e.g., half-updated indexes).

---

### 4) Downgrade prevention (mandatory)
Konnaxion MUST enforce a deterministic downgrade policy.

Default policy:
- do not activate a pack with a lower `pack_version` than the current active pack.

Downgrade is permitted only when explicitly triggered by a rollback action or operator-approved policy (e.g., emergency).

---

### 5) Deterministic rollback (mandatory)
Konnaxion MUST provide rollback to a verified pack.

Minimum supported modes:
- **Pinned rollback**: activate an explicitly pinned known-good pack.
- **Last-known-good rollback**: activate the most recently active verified pack.

Rollback behavior MUST be deterministic given the same trigger sequence and policy state.

Rollback triggers MAY include:
- explicit operator action
- verified index updates marking a release revoked
- local runtime health signals (optional profile; must be policy-defined)

---

## Consequences

### Positive
- Clients cannot activate tampered or incompatible artifacts.
- Correctness and safety boundaries are enforced at the edge.
- Recovery from regressions is deterministic and auditable.
- Offline operation remains correct: trust roots are pinned and verification can be performed locally.

### Trade-offs
- Verification adds CPU and I/O overhead (especially full file hash checks).
- Requires key management and pinned trust roots per channel.
- Requires additional storage to keep multiple versions for rollback.

---

## Alternatives considered

### A) Best-effort activation with warnings
Rejected. Silent or partial activation can corrupt correctness and makes failures non-reproducible.

### B) Network-required trust verification
Rejected. Correctness must not depend on network availability at activation time; offline operation is a core goal.

### C) Automatic downgrade on failure without explicit policy
Rejected. This can hide regressions and makes behavior non-deterministic if triggers are ambiguous.

---

## Implementation notes

- Runtime Pack manifests MUST declare:
  - `pack_id`, `pack_version`, `source_exchange_id`, `build_id`
  - `compatibility.query_contract_version`
  - `files[]` with per-file hash entries where feasible
  - optional `signatures[]` with `kid`
- Konnaxion MUST pin trust roots per channel (tenant/environment/channel).
- The on-disk storage layout SHOULD preserve multiple versions:
  - active pointer
  - last-known-good pointer
  - pinned versions
- Telemetry MUST include:
  - `PACK_VERIFY_*`, `PACK_ACTIVATION_*`, `PACK_ROLLBACK_*`
  - deterministic error codes

---

## References

- `docs/03-nodes/04-chesed-konnaxion.md`
- `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
- `docs/06-operations/03-observability.md`
