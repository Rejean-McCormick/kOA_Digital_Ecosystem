# ADR-0013: Determinism, Versioning, and Compatibility Policy

- **Status**: Accepted
- **Date**: 2026-02-09
- **Owner**: Orgo / Kristal / Konnaxion / Architect Spec
- **Related**:
  - `docs/01-principles/03-determinism.md`
  - `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
  - `docs/04-artifacts-contracts/07-render-bundle.md`
  - `docs/03-nodes/11-daat-kristal.md`
  - `docs/03-nodes/04-chesed-konnaxion.md`
  - `docs/03-nodes/08-hod-architect-render.md`
  - `docs/06-operations/02-release-management.md`

---

## Context

The ecosystem must remain “tight” across:
- build pipelines (validation → compile),
- distribution (verification → activation → rollback),
- rendering (trace/no-new-facts),
- and execution (task observability).

Without a single determinism and compatibility policy, different components may:
- interpret “latest” differently,
- activate incompatible artifacts,
- produce irreproducible outputs,
- or hide regressions via silent fallbacks.

Determinism is not applied uniformly: some nodes must be strictly deterministic; others may be heuristic but must remain auditable and bounded.

---

## Decision

## 1) Determinism is mandatory for gates and derived artifacts

### 1.1 Strict determinism REQUIRED for:
- Orgo validation gates and workflow gating decisions
- Kristal compilation outputs (Exchange + Pack manifests)
- Konnaxion verification and activation decisions
- Architect-Render outputs in deterministic mode (including trace_map ordering)

### 1.2 Determinism OPTIONAL (auditable) for:
- Architect-Strategy planning (unless deterministic mode is enabled)
- SwarmCraft execution strategies involving external systems (must be auditable; idempotency required)

In all cases: non-determinism must never cross the truth boundary.

---

## 2) Versioning and pinning policy (no floating references)

Correctness-critical paths MUST pin:
- mandate version
- blueprint version
- policy versions
- exchange_id and/or pack_id when consumed for execution/render correctness
- template id+version for rendering
- component versions + config hashes for reproducibility

Forbidden in correctness paths:
- floating references like “latest” or “current”
- implicit default policies not recorded in manifests

Allowed only for convenience surfaces:
- exploratory browsing
- non-authoritative previews clearly labeled as such

---

## 3) Identity policy for canonical artifacts

### 3.1 Preferred: content-addressed identity
When enabled:
- `exchange_id` and `pack_id` are derived from canonical serialization + pinned context + compiler config hash.
- identical inputs/config MUST produce identical artifacts and IDs.

### 3.2 Acceptable: stable ID mode
If content-addressing is not feasible:
- IDs may be stable but not purely content-derived,
- build manifests MUST bind all inputs/versions/config hashes required to reproduce intent and audit.

---

## 4) Compatibility policy (contract versions)

### 4.1 Runtime Pack compatibility
Every Runtime Pack manifest MUST declare:
- `compatibility.query_contract_version`
- optional `required_capabilities[]`
- optional runtime min/max versions

Konnaxion MUST:
- refuse activation if `query_contract_version` is unsupported
- refuse activation if required capabilities are missing
- refuse activation if projections required by the client are missing

### 4.2 Render compatibility
Render Bundles MUST declare:
- template id+version
- renderer engine version + config hash
- strictness profile
- pinned source exchange/pack IDs

Consumers may cache outputs only if:
- pinned IDs and versions match
- integrity constraints are met

---

## 5) Migration and upgrades

### 5.1 Backward compatibility window
Each contract type MUST define:
- minimum supported schema version
- deprecation policy and timeline
- migration strategy

### 5.2 Schema evolution rules
- additive fields are preferred
- breaking changes require:
  - new schema version
  - compatibility rules
  - migration guidance
  - ADR entry

---

## 6) Downgrade and rollback policy

- `pack_version` MUST be monotonic within a channel.
- Konnaxion MUST enforce downgrade prevention by default.
- Rollback is permitted only through explicit policy/trigger and must be deterministic.
- Rollback must be observable (events + reason codes).

---

## Consequences

### Positive
- Reproducible builds and renders.
- Safe offline activation with deterministic verification.
- Clear separation between authoritative vs convenience surfaces.
- Predictable upgrade and rollback behavior.

### Trade-offs
- Requires pinning discipline (more verbose metadata).
- Requires schema/version management and migration work.
- Limits “magic” convenience behaviors in correctness paths.

---

## Alternatives considered

### A) Allow floating “latest” everywhere
Rejected. Breaks reproducibility and enables inconsistent behavior between components.

### B) Implicit compatibility via best-effort parsing
Rejected. Hidden fallback behavior causes silent correctness failures.

### C) Determinism everywhere (even planning)
Rejected as a default. Planning can remain heuristic, but it must be auditable and bounded.

---

## Implementation notes

- Orgo must persist pinned versions in build and task records.
- Kristal must include pinned versions/config hashes in build manifests.
- Konnaxion must enforce compatibility using manifest fields and pinned trust roots.
- Architect-Render must output deterministic render bundles in deterministic mode.
- Observability must report versions/config hashes and reason codes at every gate.

---

## References

- `docs/01-principles/03-determinism.md`
- `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
- `docs/04-artifacts-contracts/07-render-bundle.md`
- `docs/06-operations/03-observability.md`
