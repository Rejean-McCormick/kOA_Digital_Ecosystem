# Chesed (Connectivity / Expansion) — Konnaxion

Konnaxion is the ecosystem’s outward-facing connectivity node: the “Who” layer (identity, roles, collaboration, community signals) and the distribution layer for Runtime Packs (delivery, verification, activation, rollback). It is the portal between the internal pipeline (Orgo/Kristal/Architect) and end users.

---

## 1) Scope and non-goals

### In scope
- Social facet: identities, roles, permissions, collaboration surfaces, inboxes, contribution and interaction logs.
- Distribution facet: channel-scoped delivery of Runtime Packs; verification; compatibility checks; atomic activation; rollback; offline caching.
- Telemetry and feedback signals as operational signals (non-mutating to canon).
- Trust and safety enforcement at the edge (integrity checks, privacy controls, moderation queues).

### Out of scope
- Building packs (owned by Orgo + Kristal compiler).
- Canonical truth creation or mutation (Kristal Exchange is authoritative; Konnaxion reads/distributes only).
- Internal execution orchestration (SwarmCraft owns execution).

---

## 2) Responsibilities

## 2.1 Social facet (“Who”)
- Identity and access control:
  - user profiles, roles, permissions, visibility rules
  - confidentiality settings (public / pseudonym / anonymous)
- Collaboration surfaces:
  - notifications, feeds, cross-module search integration
  - participation signals: votes, debates, comments, consultations
- Trust ledger integration (EkoH):
  - maintain expertise/ethics weights
  - compute weighted voting outcomes (Smart Vote)
  - preserve audit trails of score changes

## 2.2 Distribution facet (Runtime Pack delivery)
- Consume Orgo “Release” metadata and channel pointers.
- Store pack bundles and metadata with deterministic cache policies.
- Verify integrity and trust roots before activation (fail-closed).
- Enforce compatibility checks before activation.
- Perform atomic activation (no partial activation).
- Implement downgrade prevention and deterministic rollback.
- Support low-bandwidth and offline operation.

---

## 3) Interfaces and boundaries (contracts)

## 3.1 Primary upstream dependencies
- Kristal: Runtime Pack bundles + manifests (and optional signatures).
- Orgo: Release records, cohort targeting policies, channel pointers (active/latest/pinned/minimum).
- EkoH: expertise/ethics scores and audit trails (weighting inputs).
- Architect outputs: rendered UI payloads may be served through Konnaxion surfaces, but Konnaxion does not generate them.

## 3.2 Downstream surfaces
- End-user clients (web/mobile/desktop/offline clients).
- Public/partner APIs (where applicable).
- Moderation and admin consoles.

## 3.3 Hard boundary rules
- Konnaxion MUST NOT mutate Kristal Exchange.
- Konnaxion MUST NOT “repair” or reinterpret canon at the edge.
- Konnaxion MUST NOT activate unverified or incompatible packs.
- Feedback and telemetry MUST create governed work (Cases/Tasks) rather than altering canon.

---

## 4) Data model (minimum)

## 4.1 Distribution identifiers and required files
A distributable Runtime Pack bundle MUST include:
- `pack_manifest.json` (mandatory)
- pack payload files (indexes/tables/dictionaries/etc.)
Optional:
- signatures/hashes/kid fields
- `pack_index.json` (channel-level signed index)

Each bundle MUST carry:
- `pack_id` (preferred content-addressed identity, or deterministic content id)
- `source_kristal_id`
- `build_id`
- structured `pack_version`

## 4.2 Channels (tenant/environment scoping)
Packs are distributed via channels scoped at minimum by:
- `tenant_id`
- `environment` (`prod`, `staging`, etc.)
Optional:
- `region` or `audience_segment`

Trust roots and activation rules are channel-scoped and MUST NOT be mixed across channels unless explicitly configured.

---

## 5) Distribution lifecycle

## 5.1 Verification (fail-closed)
If a pack declares signatures/hashes/kid, Konnaxion MUST:
1) verify channel index (if used)
2) verify manifest signature(s)
3) optionally verify bundle file hashes (profile; may be expensive)
4) only then activate

Trust roots MUST be pinned per channel and verifiable offline. Trust roots MUST NOT be fetched over the network at activation time as a dependency for correctness.

## 5.2 Activation (atomic)
Konnaxion MUST only activate when:
1) integrity verified (if declared)
2) manifest parses and passes schema validation
3) required files referenced by manifest exist
4) pack is compatible with the client runtime (contract versions match)

Activation MUST be an atomic switch (no partial activation).

## 5.3 Compatibility checks (mandatory)
Before activation, Konnaxion MUST verify:
- supported `query_contract_version`
- policy selections supported by the client runtime
- required projections/data are present for consuming modules

If incompatible:
- do not activate
- emit deterministic error code + diagnostic payload

## 5.4 Downgrade prevention (mandatory)
At least one deterministic policy MUST exist:
- monotonic version rule: do not activate a lower `pack_version` than current unless explicitly permitted (e.g., emergency rollback)
- revocation-aware rule: do not activate revoked packs from a verified channel index

Rollback must be explicit (see below) and never “silent.”

## 5.5 Rollback (mandatory)
Konnaxion MUST support at least one rollback mode:
- pinned rollback (activate previously pinned known-good pack)
- last-known-good rollback (activate most recent previously active verified pack)

Rollback triggers MAY include:
- explicit operator action (recommended)
- verified index updates (e.g., “latest revoked”)
- local runtime health signals (optional profile)

Rollback MUST be deterministic given the same trigger event sequence.

## 5.6 Offline caching and storage (mandatory to define)
Konnaxion SHOULD store packs to support:
- multiple installed versions per channel
- atomic activation pointers
- garbage collection with pinning rules

Example layout:
- `/<tenant>/<env>/<channel>/packs/<pack_id>/...`
- `/<tenant>/<env>/<channel>/active -> <pack_id>`

Konnaxion MUST define deterministic cache policies:
- max disk usage per channel
- eviction policy (e.g., LRU with pinned exceptions)
- minimum pinned set to retain (active + last-known-good)

If offline, Konnaxion MUST:
- continue serving from the active pack
- not attempt activation that depends on network-fetched trust roots
- optionally surface “stale pack” metadata

---

## 6) Social + trust subsystem (EkoH / Smart Vote)

### Core trust artifacts (examples)
- Expertise domains (categories)
- Per-user expertise scores per domain (raw + weighted)
- Per-user ethics score (multiplier)
- Score configuration parameters (weights)
- Score history audit trail (immutable append-only)
- Confidentiality settings (public/pseudonym/anonymous)

### Weighted voting artifacts (examples)
- Vote (raw + weighted value)
- Vote modality (parameters)
- Vote result aggregates (sum weighted, counts)
- Integration mappings (cross-module context linkage)

### Constraints
- Access control and confidentiality must be enforced consistently across UI and APIs.
- Auditable provenance of actions is mandatory for meaningful trust weighting.
- Reputation/voting signals must not directly mutate canon; they influence governance and future work.

---

## 7) Observability (minimum)

## 7.1 Distribution telemetry (non-mutating)
Konnaxion MAY emit operational signals to Orgo such as:
- download/verification failures
- activation success/failure
- query runtime errors and performance summaries
- pack usage metrics (counts, not raw facts)

These signals MUST NOT mutate Kristal Exchange directly; they should open Cases/Tasks or adjust distribution policy.

## 7.2 Social telemetry
- audit logs: authentication, role/permission changes
- moderation queue events
- vote and score update events (with ScoreHistory)
- privacy mode transitions (confidentiality setting changes)

---

## 8) Failure modes (expected behavior)

- Verification failure: fail-closed; do not activate; emit deterministic diagnostics.
- Channel index tampering: fail-closed; ignore index; remain on active/known-good.
- Incompatibility: do not activate; emit deterministic error and keep active pack.
- Storage pressure: deterministic eviction respecting pinned packs; never evict active without replacement.
- Offline: serve active pack; queue non-critical operations; do not require network trust roots.
- Abuse/brigading/spam: flag anomalies; apply moderation rules; exclude flagged content from aggregation if policy requires.

---

## 9) Conformance tests (required)

A conformant Konnaxion implementation MUST provide tests for:
- signature verification success/failure (fail-closed)
- atomic activation (no partial state)
- downgrade prevention behavior
- rollback behavior and determinism
- offline behavior (serve active; no network dependency for correctness)
- cache eviction respects pinned packs

---

## 10) Appendix: recommended internal split

Konnaxion has two facets with different risk profiles and contracts:
- Social facet: profiles/roles/inboxes/collaboration and community signals.
- Distribution facet: delivery/verification/activation/rollback of Runtime Packs.

Treat them as separately testable subsystems even if deployed together.
