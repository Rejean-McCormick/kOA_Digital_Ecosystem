# Runtime Pack (Kristal Runtime Pack)

**File:** `docs/04-artifacts-contracts/06-runtime-pack.md`  
**Status:** Canonical (normative)  
**Owner:** Kristal (truth pivot) + Konnaxion distribution facet (activation contract)  
**Purpose:** Define the Runtime Pack artifact: identity, manifest, integrity, compatibility, activation/rollback, offline behavior, and conformance.

---

## 1) Definition

A **Runtime Pack** is a **derived, indexed, offline-executable representation** of Kristal knowledge intended for client-side or edge consumption.

### Non-goals
- Runtime Packs are **not** the canonical truth store (that is the Exchange).
- Runtime Packs are **not** a full semantic endpoint; they expose **constrained query semantics** (explicitly NOT full SPARQL).

---

## 2) Required components

A Runtime Pack MUST include:

1) **Manifest** (machine-readable, schema-validated)  
2) **Pack payload(s)** referenced by the manifest (data files, indexes, projections, etc.)  
3) Optional: signatures and integrity metadata (if present, verification is mandatory and fail-closed)

---

## 3) Identity and canonicalization

### 3.1 Canonicalization profile
Artifacts MUST declare:
- `canonicalization_profile` (e.g., `"jcs-rfc8785"`)
- `canonicalization_version` (e.g., `"1.0"`)

### 3.2 Content-addressed pack_id
Runtime Packs MUST have a stable `pack_id`.  
`pack_id` MUST be content-addressed from the pack’s declared reproducibility surface (at minimum: manifest + referenced payload hashes).

### 3.3 Determinism requirement
Given identical:
- input snapshots
- compiler version
- build configuration (`config_hash`)
- policy selections

then Runtime Pack rebuild MUST produce identical `pack_id` and identical declared payload hashes.

---

## 4) Manifest (mandatory)

### 4.1 Manifest required fields (minimum)
Every Runtime Pack MUST have a manifest containing at minimum:
- `build_id` (unique build identifier)
- `build_timestamp` (ISO 8601)
- `compiler.name` and `compiler.version`
- `config_hash` (hash of the full build configuration)
- `input_snapshots` list (content-addressed references to inputs used)
- `canonicalization_profile` + `canonicalization_version`
- `policy_selections` (portable policy identifiers)
- declared hashes for the artifact payload(s)

### 4.2 Schema and parsing
- The manifest MUST pass schema validation before any activation attempt.
- Unknown **non-integrity** fields SHOULD be ignored for forward compatibility.
- Integrity fields are never best-effort.

---

## 5) Integrity and signatures (fail-closed)

### 5.1 Signature envelope rule
If signatures exist, they MUST be in a separated envelope so signature material can be removed deterministically before hashing.

### 5.2 Hash/sign workflow
Signing and verification MUST follow:
1) Remove signature material (if present)  
2) Canonicalize  
3) Hash (SHA-256)  
4) Verify and/or attach signatures

### 5.3 Fail-closed semantics
If a pack declares **any** of:
- hashes
- signatures
- signer identity / key reference (`kid`)

then verifiers MUST:
- fail closed if verification fails
- fail closed if integrity metadata is malformed or ambiguous
- fail closed if declared hash does not match computed hash

---

## 6) Query contract (runtime semantics)

Runtime Packs are optimized for offline use and MUST declare a **query contract version** (or equivalent) that the consuming runtime can check.

### 6.1 Constrained query semantics
- Packs MUST define supported query patterns explicitly.
- Packs MUST NOT claim full SPARQL parity.

### 6.2 Join-cap and constrained execution (recommended)
If the runtime enforces join caps or other constraint controls, those constraints SHOULD be declared and testable (e.g., strict join-cap error codes).

---

## 7) Compatibility checks (mandatory before activation)

Before activation, the client MUST verify:
- supported `query_contract_version`
- supported `policy_selections` (otherwise reject deterministically)
- required projections/data files exist for consuming modules

If incompatible:
- do not activate
- emit deterministic error code + diagnostics payload

---

## 8) Distribution and activation (Konnaxion contract)

### 8.1 Trust roots (mandatory)
Konnaxion MUST pin trust roots per channel (tenant/environment) and MUST be able to verify offline.  
Trust roots MUST NOT be fetched over the network at activation time as a dependency for correctness.

### 8.2 Activation rule (mandatory)
Konnaxion MUST only activate a pack when:
1) integrity is verified (if declared)  
2) manifest parses and passes schema validation  
3) required files referenced by manifest exist  
4) pack is compatible with the client runtime

Activation MUST be an atomic switch (no partial activation).

### 8.3 Channel index (recommended)
Konnaxion SHOULD consume a signed channel index (e.g., `pack_index.json`) with:
- `channel_id`
- `latest` pointer(s)
- `pinned` pointer(s)
- optional `minimum_allowed_version`
- optional `revoked` list

If present and signed, the index MUST be verified using the channel’s trust roots (fail-closed).

---

## 9) Rollback and downgrade prevention (mandatory)

### 9.1 Rollback modes
Konnaxion MUST support at least one:
- **Pinned rollback** (activate a pinned known-good pack)
- **Last-known-good rollback** (activate most recent previously active pack still present and verified)

### 9.2 Rollback determinism
Rollback MUST be deterministic given the same trigger event sequence.

### 9.3 Downgrade prevention
Konnaxion MUST implement downgrade prevention in at least one deterministic form:
- monotonic version rule (recommended)
- revocation-aware rule (do not activate revoked packs)

If rollback is permitted, it MUST occur via explicit rollback action, not silently.

### 9.4 Minimum version pin (recommended)
Maintain per-channel `min_allowed_release_version`.  
Clients MUST NOT activate packs below this minimum (even if cached).  
If a pack is discovered vulnerable/invalid: revoke it, distribute revocation list, bump minimum pin to exclude it.

---

## 10) Offline caching and storage contract

### 10.1 Activation is local and asynchronous
Devices may download later than publication and activate later than download.  
Integrity verification MUST be performed at activation time.  
Store the activation decision with an audit marker (`pack_id`, `release_id`, timestamp).

### 10.2 Storage constraints
If devices cannot hold Blue+Green simultaneously:
- retain at least the previous manifest + signatures, OR
- implement a checkpoint strategy for last-known-good payloads where feasible.

### 10.3 Storage layout (recommended)
Konnaxion SHOULD store packs in a layout that supports:
- multiple installed versions per channel
- atomic activation
- garbage collection with pinning rules

Example layout:
- `/<tenant>/<env>/<channel>/packs/<pack_id>/...`
- `/<tenant>/<env>/<channel>/active -> <pack_id>`

### 10.4 Cache policy (mandatory to define)
Konnaxion MUST define deterministic cache policies:
- max disk usage per channel
- eviction policy (e.g., LRU with pinned exceptions)
- minimum pinned set (e.g., active + last-known-good)

---

## 11) Release strategies (recommended)

### 11.1 Blue/Green (default)
Maintain two slots per channel:
- Blue: active
- Green: candidate

Verify at rest and run pack-local smoke tests before switching the channel pointer Blue → Green. Retain Blue for rollback window.

### 11.2 Canary (high-risk)
Roll out to deterministic cohorts (e.g., `hash(device_id) mod 100`) and promote gradually.  
Abort and rollback on health failures.

---

## 12) Telemetry and feedback (non-mutating)

Konnaxion MAY emit operational signals to Orgo such as:
- download/verification failures
- activation success/failure
- query runtime errors and performance summaries
- pack usage metrics (counts)

These signals MUST NOT mutate Exchange directly; they should create Cases/Tasks or distribution adjustments.

---

## 13) Conformance tests (required)

A conformant Runtime Pack ecosystem MUST include tests for:
- manifest schema validation
- pack_id determinism (rebuild = same pack_id/hashes)
- fail-closed verification on tamper/malformed integrity fields
- atomic activation (fully active or not)
- compatibility rejection with deterministic error codes
- deterministic rollback behavior
- downgrade prevention enforcement

---

## 14) Minimal example (illustrative)

A minimal manifest example (illustrative fields only):

```json
{
  "build_id": "bld_2026_02_09_0001",
  "build_timestamp": "2026-02-09T12:00:00Z",
  "compiler": { "name": "kristal-compiler", "version": "3.0.0" },
  "config_hash": "sha256:...",
  "input_snapshots": ["sha256:...", "sha256:..."],
  "canonicalization_profile": "jcs-rfc8785",
  "canonicalization_version": "1.0",
  "policy_selections": ["policy:baseline"],
  "payloads": [
    { "path": "data.parquet", "hash": "sha256:..." },
    { "path": "index.bin", "hash": "sha256:..." }
  ],
  "query_contract_version": "1"
}
