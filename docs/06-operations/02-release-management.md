# Release Management

**File:** `docs/06-operations/02-release-management.md`  
**Status:** Canonical (normative for production; recommended sections marked)  
**Purpose:** Define how Runtime Packs are released, promoted, activated, monitored, and rolled back safely across channels, tenants, and offline clients.

---

## 1) Scope and concepts

### 1.1 Objects
- **Build**: produces `kristal_id` + `pack_id` with manifests/hashes/signatures.
- **Release**: binds a build/pack to deployment parameters (channel, cohorts, environment), enabling traceability build → release → distribution.
- **Channel Pointer / Index**: defines what release is current for a channel (and optionally for cohorts).
- **Device/Node State**: records what is installed, verified, and active.

### 1.2 Required properties
- Releases must be auditable and reproducible.
- Activation must be gated by verification and local smoke tests.
- Rollbacks must be explicit and controlled (downgrade prevention is default).

---

## 2) Roles and responsibilities

### 2.1 Orgo (control plane)
Orgo MUST:
- create and version Release objects;
- bind each Release to a known Build and `pack_id`;
- record channel/cohort parameters for traceability;
- coordinate promotion/abort decisions based on health signals.

### 2.2 Konnaxion (distribution facet)
Konnaxion MUST:
- distribute pack payloads to the right targets;
- verify manifests/hashes/signatures before activation (fail-closed when declared);
- perform atomic activation;
- implement rollback and downgrade policy locally for offline safety.

---

## 3) Default release strategy (recommended): Blue/Green

### 3.1 Concept
Maintain two production slots per channel:
- **Blue**: current active pack version
- **Green**: new candidate pack version

### 3.2 Procedure (normative)
1) **Publish Green**
   - Orgo creates a Release referencing the new `pack_id`.
   - Distribution replicates pack payloads to the Green slot.

2) **Verify at rest (pre-activation)**
   - Verify manifest hash, payload hashes, signatures (if present).
   - If verification fails: do not activate; emit a deterministic failure.

3) **Warm-up / readiness checks**
   - Run pack-local smoke tests before activation:
     - basic query smoke tests,
     - join-cap behavior (if enforced),
     - latency threshold checks,
     - memory footprint checks.
   - Log outcomes with correlation identifiers.

4) **Switch**
   - Flip the channel pointer from Blue → Green.
   - Retain Blue for a defined rollback window.

5) **Post-switch monitoring**
   - Observe error codes, latency, resource usage.
   - If thresholds exceed: trigger rollback.

### 3.3 Pitfalls (operational guidance)
- Some split-brain (some devices on Blue, some on Green) is expected in offline/asynchronous environments; what matters is auditability and deterministic activation rules.

---

## 4) High-risk strategy: Canary releases

### 4.1 Concept
Roll out a new pack to a small cohort first, validate, then expand.

### 4.2 Cohort assignment (normative)
Cohorts MUST be stable and deterministic (e.g., `hash(device_id) mod 100`).

### 4.3 Procedure (normative)
1) Create cohorts (e.g., 1% → 10% → 50% → 100%)
2) Canary publish: Orgo creates a Release for cohort 1%
3) Activation gating: devices verify integrity and run smoke tests before activation
4) Promote: expand to next cohort after an observation window if metrics are healthy
5) Abort and rollback: stop promotion and revert cohorts to previous pack pointer if health fails

### 4.4 Metrics to watch (required)
- integrity verification failures
- query error rates (especially constraint errors such as join-cap)
- median/P95 query latency
- memory pressure / OOM events
- pack download failures or pack size complaints

---

## 5) Activation gating (mandatory)

Before activation, clients/devices/nodes MUST:
1) parse and validate the pack manifest;
2) verify declared hashes/signatures (if present);
3) confirm compatibility (query contract version, required projections);
4) run minimal smoke tests (pack-local);
5) only then perform **atomic activation**.

Activation MUST be all-or-nothing; partial activation is forbidden.

---

## 6) Rollback and downgrade safety (critical)

### 6.1 Default rule: block downgrades
Clients MUST persist per-channel highest-seen/highest-activated release information and MUST block activating lower release IDs by default.

### 6.2 Minimum version pin (mandatory)
Maintain per-channel `min_allowed_release_version`.
- Clients MUST NOT activate packs below this minimum, even if cached.
- Operators MUST bump the minimum pin when invalid/vulnerable packs are discovered.

### 6.3 Revocation (mandatory)
If a pack or signing key is discovered invalid/vulnerable:
- mark it revoked in release metadata,
- distribute a signed revocation list artifact,
- bump the minimum pin to exclude it.

### 6.4 Controlled rollback (policy-authorized rollback)
A rollback MAY be permitted only with an explicit signed authorization record.
Authorization SHOULD include:
- `channel_id`
- `authorized_rollback_to_release_id`
- `reason`
- `issued_at`, `expires_at`
- signature by an authority key

When rollback authorization is present, clients MAY activate the rollback target even if lower than previously active, but MUST:
- keep `highest_seen_release_id` unchanged,
- record `current_active_release_id` separately (recommended),
- record an audit event that rollback occurred.

### 6.5 Offline edge cases (mandatory safe behavior)
If offline and only older artifacts are available:
- clients MUST NOT downgrade without valid cached rollback authorization;
- clients SHOULD enter a “hold” state rather than silently downgrade.

---

## 7) Signing, trust roots, and rotation (mandatory in production)

### 7.1 What is signed (normative)
- Runtime Pack signing target MUST be unambiguous and recorded in the manifest.
- Verification MUST validate canonical hashing rules and enforce validity windows.

### 7.2 Trust root pinning (mandatory)
Pins (trust root fingerprints) MUST be distributed out-of-band or via secure bootstrap and MUST be stored locally to enable offline verification.
Pin sets SHOULD support multiple active roots to enable rotation.

### 7.3 Key rotation (required policy)
- Trust roots rotate rarely; intermediates periodically; signing keys frequently.
- Rotation MUST support overlap windows (old+new valid simultaneously).
- Compromise response MUST revoke affected key IDs, stop issuing artifacts with compromised keys, and re-issue current packs with new keys.

### 7.4 Revocation lists (offline-friendly)
Use signed revocation list artifacts (e.g., `revocations.json`) distributed alongside packs and/or via sync channels.
Clients MUST consult the latest available revocation list before accepting signatures when revocation checking is required by policy; fail-closed if required and missing.

---

## 8) Control-plane data model (recommended)

A practical release system maintains at minimum:
- `BuildRecord(build_id, pack_id, kristal_id, hashes, signatures, policies, inputs, compiler_version, config_hash)`
- `Release(release_id, channel_id, pack_id, signing_key_id, published_at, status, cohort_spec?)`
- `ChannelPointer(channel_id, active_release_id, min_allowed_release_version)`
- `DeviceState(device_id, channel_id, active_pack_id, last_verified_at, verification_status, highest_seen_release_id, highest_activated_release_id?)`

---

## 9) Observability and audit (required)

### 9.1 Required event types (recommended)
Emit structured events such as:
- `PACK_VERIFIED` (artifact_id, release_id, signer_key_id)
- `PACK_REJECTED` (verification_failed | downgrade_blocked | revoked_key | index_signature_failed)
- `DOWNGRADE_BLOCKED`
- `ROLLBACK_AUTH_ACCEPTED` / `ROLLBACK_AUTH_REJECTED`
- `PACK_ACTIVATED` (active_release_id, highest_seen_release_id)

### 9.2 Correlation identifiers (required)
Events SHOULD include:
`build_id`, `kristal_id`, `pack_id`, `release_id`, `channel_id`, `tenant_id`, and `device/node_id`.

---

## 10) Conformance checklist (required)

A conformant release-management implementation MUST demonstrate:
- Blue/Green or equivalent safe promotion pattern
- deterministic cohort assignment for canary (if used)
- activation gating: schema validation + verification + smoke tests + atomic activation
- downgrade prevention by default
- minimum version pin enforcement
- revocation list distribution and enforcement per policy
- rollback authorization requirement for controlled rollbacks
- offline-safe behavior (hold > silent downgrade)
- structured telemetry and audit linkage build → release → distribution → activation
