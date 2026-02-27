# Release Record (kOA-native artifact)

**File:** `docs/30-artifacts/release-record.md`  
**Normative scope:** kOA-native artifact contract (control-plane).  
**Non-normative:** Kristal artifact schemas and cryptographic/canonicalization mechanics (external).

---

## 1) Purpose

A **Release Record** is the kOA control-plane artifact that captures **distribution intent** and **rollout state** for publishing one or more verified build outputs (e.g., Runtime Packs) to one or more channels/cohorts.

It binds:
- **what** is being released (references to build outputs),
- **where** it is being released (channels/cohorts),
- **how** it is being released (policy + rollout strategy),
- **what happened** (verification/activation outcomes, timestamps, operators, reasons).

A Release Record does not mutate canonical truth. It is an operational governance artifact used by Orgo/Konnaxion.

---

## 2) Producer / consumer

- **Produced by:** Orgo (release controller)  
- **Consumed by:** Konnaxion distribution facet, rollout tooling, ops/audit systems

---

## 3) Identity and immutability

- `release_id` is the primary identifier.
- Release Records are append-only in audit stores; state transitions are recorded as events.

---

## 4) Minimal fields (conceptual contract)

### 4.1 Core identifiers
- `release_id` (string, unique)
- `created_at` (RFC3339 timestamp)
- `created_by` (principal/user/service identity)

### 4.2 What is being released
- `build_ref` (content ref / ID pointing to the Build Record)
- `artifacts[]` (list of artifact refs; typically Runtime Pack refs; may include additional derived artifacts)
  - each item includes:
    - `artifact_type`
    - `artifact_ref` (content ref / ID)
    - `artifact_hash` (hash object, if applicable)
    - `compat` (optional compatibility metadata)

### 4.3 Where it is being released
- `channels[]` (e.g., `stable`, `beta`, `canary`, tenant-scoped channels)
  - each channel entry includes:
    - `channel_id`
    - `target_cohorts[]` (optional)
    - `constraints` (optional policy constraints)

### 4.4 Rollout strategy
- `strategy` (object)
  - examples: `all_at_once`, `percentage`, `staged`, `pinned`
- `rollout_plan` (optional; stages with conditions)

### 4.5 Verification + activation policy hooks
- `verification_policy_ref` (policy identifier/ref)
- `activation_policy_ref` (policy identifier/ref)

### 4.6 Current state
- `status` (enum): `DRAFT | QUEUED | VERIFYING | VERIFIED | RELEASING | RELEASED | PAUSED | ROLLED_BACK | FAILED | CANCELED`
- `status_reason` (optional text/code)
- `updated_at` (RFC3339 timestamp)

### 4.7 Events (append-only)
- `events[]` ordered list of state transitions / actions
  - each event includes:
    - `at` timestamp
    - `type` (e.g., `QUEUED`, `VERIFY_STARTED`, `VERIFY_FAILED`, `ACTIVATE_SUCCEEDED`, `ROLLBACK_SUCCEEDED`)
    - `actor`
    - `details` (structured)

---

## 5) Invariants (kOA)

- Release Records MUST be produced only by Orgo-controlled workflows.
- Activation MUST be fail-closed: if verification fails, `RELEASED` must never occur.
- Rollback MUST be traceable and recorded as events.
- Release Records MUST never rewrite canonical artifacts; they only reference them.

---

## 6) Failure modes

- Missing or unverifiable artifact refs → `FAILED`
- Incompatible runtime pack vs target cohort constraints → `PAUSED` or `FAILED` (policy-dependent)
- Partial rollout failures → `PAUSED` with cohort-specific event details

---

## 7) Storage and access

- Stored in Orgo audit store (append-only).
- Exposed via Orgo API for operators and automation.

---

## 8) Schema

- JSON Schema: `docs/30-artifacts/schemas/release-record.schema.json`

---