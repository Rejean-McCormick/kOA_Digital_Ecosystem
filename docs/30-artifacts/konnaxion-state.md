# Konnaxion State (kOA-native Operational Artifact)

## Purpose
**Konnaxion State** is a **kOA-native operational artifact** that captures the local distribution/activation state of a Konnaxion instance (or cluster) in a structured, auditable form.

It is designed to:
- provide operators and Orgo a consistent view of what is installed/active/pinned,
- support deterministic rollback decisions,
- correlate activation outcomes with health signals.

This artifact does **not** redefine Kristal artifacts (Exchange, Runtime Pack, Validation Report). It references them by ID as defined in the pinned Kristal v4 spec. See `docs/40-integration/kristal-v4/`.

---

## Ownership
- **Produced by:** Konnaxion
- **Consumed by:** Orgo (ops + audit), rollout tooling, monitoring/telemetry pipelines
- **Storage:** local state store + optional upstream audit store

---

## When it is emitted
Konnaxion MUST emit an updated Konnaxion State record on:
- successful verify+activate
- failed verification or failed activation attempt
- rollback start and rollback completion
- pin/unpin or revocation changes
- periodic heartbeat (recommended)

---

## Identifier
- `state_id`: unique ID for the record (UUID recommended)
- `instance_id`: stable identifier of the Konnaxion instance emitting the record
- `created_at`: RFC3339 UTC timestamp

---

## Semantics (normative)

### Active set invariants
Konnaxion State MUST represent:
- exactly one **active** Runtime Pack per activation domain (e.g., per channel, per tenant, per environment), according to your rollout model
- the complete set of **installed** packs available for activation
- any **pinned** or **revoked** pack constraints that affect activation decisions

### Fail-closed status
If verification cannot be completed (missing metadata, signature failure, compatibility uncertainty), the state MUST reflect a fail-closed outcome:
- `last_attempt.status = "FAILED"`
- `last_attempt.reason_code` populated
- no change to active pack unless explicitly permitted by policy

### Deterministic rollback
Rollback decisions should be derivable from:
- `active_pack` + `last_known_good_pack`
- policy pins and revocations
- deterministic triggers (health thresholds, explicit operator action)

---

## Data model (fields)

### 1) Top-level fields
- `state_id`
- `instance_id`
- `environment` (e.g., prod/stage/dev)
- `created_at`
- `active` (object; current active pack(s))
- `installed[]` (objects; installed packs)
- `pins` (object; pin/revoke constraints)
- `last_attempt` (object; most recent verify/activate/rollback attempt)
- `health` (object; summarized health signals at time of emission)
- `telemetry_refs` (object; pointers to detailed logs/traces)

### 2) Pack references
All pack references use Kristal identifiers as defined in the pinned Kristal v4 spec:
- `pack_id`
- `pack_manifest_ref` (content reference)
- `exchange_ref` (if relevant to provenance)
- `content_hash` / `signature_ref` as declared by Kristal artifacts

The exact field names and meaning are defined by Kristal; this artifact only stores references.

---

## Example (illustrative)

```json
{
  "state_id": "6d4d1a3e-9e78-4f5c-8f8f-4b3c3e1f2c11",
  "instance_id": "konnaxion-prod-a-03",
  "environment": "prod",
  "created_at": "2026-02-27T14:22:13Z",

  "active": {
    "channel": "stable",
    "pack_id": "pack_2026_02_27_0007",
    "activated_at": "2026-02-27T14:19:02Z",
    "exchange_ref": "kristal_exchange_ref_placeholder"
  },

  "installed": [
    {
      "pack_id": "pack_2026_02_27_0007",
      "installed_at": "2026-02-27T14:10:00Z",
      "manifest_ref": "contentref:sha256:...",
      "status": "VERIFIED"
    },
    {
      "pack_id": "pack_2026_02_26_0004",
      "installed_at": "2026-02-26T10:12:00Z",
      "manifest_ref": "contentref:sha256:...",
      "status": "AVAILABLE"
    }
  ],

  "pins": {
    "pinned_pack_id": "pack_2026_02_27_0007",
    "revoked_pack_ids": ["pack_2026_02_20_0011"]
  },

  "last_attempt": {
    "type": "ACTIVATE",
    "target_pack_id": "pack_2026_02_27_0007",
    "status": "SUCCEEDED",
    "reason_code": "OK",
    "started_at": "2026-02-27T14:18:50Z",
    "finished_at": "2026-02-27T14:19:02Z"
  },

  "health": {
    "status": "HEALTHY",
    "signals": [
      { "name": "startup_ok", "value": true },
      { "name": "query_smoke_ok", "value": true }
    ]
  },

  "telemetry_refs": {
    "logs": "logref:...",
    "trace": "traceref:..."
  }
}
````

---

## Validation

Schema: `docs/30-artifacts/schemas/konnaxion-state.schema.json`

---

## Change control

This is a kOA-native contract. Changes require:

* an ADR in `docs/90-reference/adr/`
* a schema version bump per your compatibility policy


