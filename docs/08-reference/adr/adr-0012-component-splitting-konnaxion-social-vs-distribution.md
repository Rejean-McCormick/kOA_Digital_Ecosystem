# ADR-0012: Component Splitting — Konnaxion Social vs Distribution

- **Status**: Accepted
- **Date**: 2026-02-09
- **Owner**: Konnaxion Spec
- **Related**:
  - `docs/03-nodes/04-chesed-konnaxion.md`
  - `docs/08-reference/adr/adr-0007-konnaxion-verification-and-rollback.md`
  - `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
  - `docs/06-operations/03-observability.md`
  - `docs/01-principles/01-invariants.md`

---

## Context

Konnaxion currently spans two distinct responsibilities:

1) **Social/Community surfaces**
- identity, roles, permissions
- inbox/notifications, participation, moderation
- feedback capture and triage routing signals
- trust/ledger integration (EkoH) signals

2) **Distribution/Edge runtime**
- Runtime Pack download, caching, verification
- compatibility checks
- atomic activation and rollback
- offline behavior and storage policy

These two facets have different:
- threat models (privacy/abuse vs integrity/tamper resistance),
- SLOs (UX latency vs activation correctness),
- data shapes (social graphs vs pack manifests/files),
- failure modes (moderation load vs verification failure spikes),
- compliance requirements (confidentiality vs integrity guarantees).

Keeping them coupled risks:
- cross-contamination of contracts (social behavior impacting activation correctness),
- expanded blast radius (a social incident affecting distribution safety),
- ambiguous ownership (who owns “fail-closed” decisions),
- and muddled documentation/testing (hard to prove conformance).

---

## Decision

### 1) Split Konnaxion into two explicitly specified subsystems

#### A) Konnaxion-Social (Chesed: “Who / Surfaces”)
Contract responsibilities:
- identity/roles/permissions/visibility
- collaboration surfaces (feeds, inboxes, moderation queues)
- feedback capture → normalized Feedback Events
- EkoH integration inputs/outputs (scores, voting aggregates), with audit trails

Hard rules:
- MUST NOT mutate Kristal Exchange
- MUST NOT activate or manage Runtime Packs
- MUST treat truth-impacting changes as governed work (Cases/Tasks)

Primary outputs:
- Feedback Events and triage-ready records to Orgo
- UI/state payloads for social surfaces
- Trust/participation signals (non-mutating)

#### B) Konnaxion-Distribution (Edge: “Packs / Activation”)
Contract responsibilities:
- Runtime Pack storage, caching, indexing (channel-scoped)
- verification-before-activation (fail-closed)
- compatibility checks before activation
- atomic activation pointer updates
- deterministic rollback (pinned, last-known-good)
- offline correctness (pinned trust roots)

Hard rules:
- MUST NOT interpret or modify canon
- MUST NOT activate unverified or incompatible packs
- MUST be deterministic given the same manifest/index/policy state

Primary outputs:
- activation state pointers per channel
- verification/activation/rollback telemetry events
- deterministic diagnostics and reason codes

---

### 2) Treat the split as a contract boundary (even if deployed together)

Even if Konnaxion-Social and Konnaxion-Distribution are deployed in a single service/process, they MUST be:
- separately testable,
- separately observable,
- separately documented,
- and independently versioned at the contract level.

---

### 3) Define the interface between the two subsystems (minimal coupling)

Allowed shared inputs:
- `tenant_id`, `environment`, `channel_id` (or equivalent)
- policy references (visibility policy vs activation policy) as immutable config refs

Disallowed coupling:
- social signals MUST NOT directly influence activation decisions
- distribution telemetry MUST NOT directly update reputation/ethics scores

If cross-signals are desired:
- they must pass through Orgo as governed work (Cases/Tasks) or through explicitly documented non-mutating analytics channels.

---

## Consequences

### Positive
- Reduced blast radius: social incidents cannot compromise pack activation integrity.
- Cleaner security posture: privacy/abuse controls separate from tamper/verification controls.
- Stronger conformance proof: “fail-closed activation” tests are isolated and auditable.
- Clearer operational ownership and incident response.

### Trade-offs
- More integration plumbing (two subsystems, explicit interface).
- Requires duplicated/common configuration patterns (tenant/channel policy scoping).
- Potentially more complex deployment (two services) if physically separated.

---

## Alternatives considered

### A) Keep Konnaxion as a single monolith
Rejected. Contracts and threat models conflict; correctness and safety boundaries become harder to prove.

### B) Partial split (modules inside one codebase, no contract boundary)
Rejected. Without an explicit contract boundary, drift and coupling re-emerge; tests become less meaningful.

### C) Split by UI vs backend only
Rejected. The real fault line is not UI vs backend; it is **social/trust surfaces** vs **distribution/activation correctness**.

---

## Implementation notes

- Documentation:
  - Keep the node doc as the conceptual node (`docs/03-nodes/04-chesed-konnaxion.md`) but explicitly describe two facets.
  - Place distribution-specific rules under ADR-0007 and Release/Observability docs.
- Data/storage:
  - Konnaxion-Social and Konnaxion-Distribution should not share write access to the same tables/collections where possible.
- Telemetry:
  - Konnaxion-Distribution must emit: `PACK_VERIFY_*`, `PACK_ACTIVATION_*`, `PACK_ROLLBACK_*`
  - Konnaxion-Social must emit: `FEEDBACK_*`, moderation events, privacy mode transitions
- APIs:
  - Konnaxion-Distribution exposes: `get_active_pack(channel)`, `list_installed_packs(channel)`, `activate(pack_id)`, `rollback(mode)`
  - Konnaxion-Social exposes: identity/roles, inbox/feed endpoints, feedback submission/triage surfaces

---

## References

- `docs/03-nodes/04-chesed-konnaxion.md`
- `docs/08-reference/adr/adr-0007-konnaxion-verification-and-rollback.md`
- `docs/04-artifacts-contracts/schemas/runtime-pack-manifest.schema.json`
- `docs/06-operations/03-observability.md`
