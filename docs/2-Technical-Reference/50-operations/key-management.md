# Key Management

## Purpose

Define how the kOA Digital Ecosystem manages cryptographic keys used for:
- signing and verifying distribution artifacts (e.g., channel indexes, activation metadata),
- authenticating internal services,
- enforcing fail-closed activation and deterministic rollback.

Kristal-owned artifact signing targets and signature field shapes are defined in the pinned Kristal v4 dependency. This document defines **kOA operational policy**: where keys live, how they rotate, how trust is anchored, and how verification gates behave.

## Scope

This doc covers:
- key roles and trust boundaries
- key storage and access control
- rotation and revocation
- verification policy at activation time
- auditing and incident response tie-ins

Not in scope:
- Kristal schema definitions
- exact signature object shapes for Kristal artifacts (refer to pinned Kristal v4)

## Key roles

### 1) Root trust (offline)
- Purpose: anchor trust for distribution verification (e.g., signing intermediate keys).
- Storage: offline HSM or equivalent.
- Access: highly restricted; dual control recommended.

### 2) Distribution signing keys (online)
- Purpose: sign channel indexes or release metadata used by Konnaxion to decide what to fetch/activate.
- Storage: online HSM / KMS.
- Rotation: frequent compared to root (e.g., 30–180 days by policy).

### 3) Service identity keys (online)
- Purpose: authenticate kOA services (Orgo, Konnaxion, pipeline components).
- Storage: workload identity platform or KMS-backed keys.
- Rotation: automated.

### 4) Developer/test keys (non-production)
- Purpose: local development and CI test signing.
- Storage: repository secrets / ephemeral CI secrets.
- MUST NOT be trusted by production verification policy.

## Key identifiers

kOA MUST use stable key identifiers in operational metadata:
- `key_id` (preferred terminology across the ecosystem)
- key metadata includes algorithm family and usage constraints
- key identifiers must be unique within the authority domain

If an external ecosystem uses `kid`, treat it as an alias mapping to `key_id` in kOA operational tooling.

## Trust store and verification policy

### Trust store contents
Konnaxion and verification services MUST maintain a trust store with:
- trusted root(s)
- allowed intermediate signer keys
- key status (active, rotated, revoked)
- validity windows and constraints

### Fail-closed verification
Activation and distribution decisions MUST be fail-closed:
- unknown key → reject
- revoked key → reject
- signature missing when required → reject
- signature invalid → reject
- algorithm not allowed by policy → reject
- key usage mismatch (wrong purpose) → reject

### Deterministic rollback coupling
If activation fails verification, the system MUST:
- preserve the last-known-good active pack state
- record a deterministic rollback decision (reason + evidence)
- emit an operational event for Orgo

## Rotation policy

### Rotation requirements
- Rotation MUST be planned and periodic for online keys.
- Rotation MUST be supported without downtime.
- Multiple valid keys MAY be accepted during an overlap window.

### Overlap window (recommended)
- Configure Konnaxion to accept signatures from:
  - the “current” key
  - the “previous” key
during a short overlap to allow safe rollout.

### Rotation procedure (high level)
1. Generate new key under controlled environment.
2. Publish the new key to the trust store with status `PENDING`.
3. Sign next release metadata with both current and new key (if supported) or switch signing to new key.
4. Promote new key to `ACTIVE` after verification in staging/limited cohort.
5. Demote old key to `DEPRECATED` and then `RETIRED`.
6. Update documentation and ADR if policy changes.

## Revocation policy

Revocation MUST be supported for:
- key compromise
- unauthorized signing
- policy violation
- cryptographic deprecation

Revocation actions:
- mark key `REVOKED` in trust store
- push trust store update to all verifiers
- block activation and downloads relying on revoked keys
- trigger rollback if an active pack’s trust chain becomes invalid (policy decision)

## Storage and access control

- Production keys MUST be stored in an HSM/KMS or equivalent.
- Keys MUST never be stored in plaintext on disk.
- Private keys MUST be accessible only to the minimum set of signing services.
- Signing services MUST authenticate to KMS with least privilege.
- All key usage MUST be logged (who, when, what was signed).

## Audit and observability

kOA MUST record:
- key creation events
- key rotation events (old→new mapping)
- signing events (artifact type, digest, signer key_id)
- verification failures (deterministic error code + reason)
- revocation events and propagation status

These events should feed:
- security monitoring
- incident response runbooks
- Orgo Cases/Tasks for remediation work

## Incident response hooks

Key compromise response SHOULD:
1. Revoke compromised key(s).
2. Freeze releases and distribution updates until trust restored.
3. Roll back affected deployments (fail-closed).
4. Create Orgo Case(s) for investigation and remediation.
5. Rotate affected trust roots/intermediates as required.

## Related docs

- `docs/50-operations/rollback.md`
- `docs/50-operations/releases.md`
- `docs/40-integration/kristal-v4/conformance.md`
- `docs/90-reference/adr/adr-0003-konnaxion-activation-rollback.md`