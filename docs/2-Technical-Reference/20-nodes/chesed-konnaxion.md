# Chesed: Konnaxion

**Role:** Connectivity + distribution + offline platform boundary.
**Primary responsibility:** safely deliver **verified, pinned** knowledge packs to products and services, and provide deterministic access patterns without mutating canonical truth.

## Responsibilities

* Fetch, cache, and serve **Runtime Packs** and related indexes/channels.
* Verify pack integrity **fail-closed** before activation (signatures, hashes, compatibility).
* Activate packs atomically and support deterministic rollback to a last-known-good or pinned version.
* Provide local query and retrieval interfaces over active packs (offline-first).
* Enforce distribution policy (channels, cohorting, pinning, revocation, downgrade prevention).
* Emit telemetry and operational signals (activation outcomes, verification failures, cache health).

## Inputs

* Release intent / rollout directives (channel, cohort, pin rules).
* Candidate pack artifacts and metadata (manifest + payloads + signatures).
* Verification keys and trust policy (key sets, allowed signers, required checks).
* Compatibility policy (supported schema versions, feature flags, migration rules).

## Outputs

* Active pack selection (the currently activated pack ID / version).
* Activation / rollback records (local operational evidence).
* Verification reports (why a candidate was rejected).
* Telemetry signals for Orgo (distribution health, failures, drift).

## Invariants

* **Fail-closed verification:** no activation unless all required checks pass.
* **Atomic activation:** a pack becomes active in one switch; partial activation is impossible.
* **Deterministic rollback:** given the same triggers and state history, rollback target selection is deterministic.
* **Downgrade prevention:** policy can forbid activating older/unsafe versions even if signatures are valid.
* **No truth mutation:** Konnaxion never edits canonical truth; it only distributes and serves verified artifacts.
* **Pinned determinism:** the same active pack + same query yields the same results (within the defined query interface).

## Failure modes

* Network/source unavailable (cannot fetch updates)
* Verification failure (signature mismatch, hash mismatch, untrusted signer)
* Compatibility failure (unsupported schema/features)
* Cache corruption / partial download
* Rollback loops (repeated activation failures)
* Key rotation mishandling (valid pack rejected due to stale trust set)

## Observability

* Pack fetch latency and failure rate
* Verification pass/fail counts by reason
* Activation success rate and time-to-activate
* Rollback frequency and triggers
* Cache utilization, corruption detection, disk pressure
* “Active pack drift” (unexpected active pack changes)

## Interfaces

### Upstream

* Distribution channels / artifact stores
* Orgo release controller (rollout directives)
* Key management service (trusted signers, revocations)

### Downstream

* Product runtimes (offline query, lookup, retrieval)
* UI services (navigation/search backed by pack indexes)
* Audit/telemetry pipeline (activation + verification evidence)

## Minimal contract (conceptual)

Konnaxion must expose:

* `get_active_pack()` → active pack identifier + metadata
* `activate(pack_ref, policy)` → success/failure + reasons
* `rollback(target_policy)` → selected target + evidence
* `query(interface, params)` → deterministic results over active pack
* `health()` → cache + verification + storage status

## Related docs

* `docs/40-integration/kristal-v4/koa-profile.md`
* `docs/50-operations/releases.md`
* `docs/50-operations/rollback.md`
* `docs/30-artifacts/konnaxion-state.md`
