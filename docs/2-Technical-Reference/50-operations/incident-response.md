# Incident Response

**File:** `docs/50-operations/incident-response.md`  
**Status:** Normative (kOA)  
**External normative references:** Kristal v4 docs + schemas (pinned dependency)

---

## 1) Purpose

Provide a deterministic, auditable procedure for detecting, triaging, mitigating, and resolving incidents across:
- Orgo build pipeline (validation/compile/publish),
- Konnaxion distribution/activation/rollback,
- Architect rendering (trace/no-new-facts),
- SwarmCraft execution (side effects + telemetry),
- Trust roots / keys / revocations,
- Multi-tenant isolation.

This runbook avoids schema-level duplication. When you need artifact formats, follow the pinned Kristal references (integration section) or kOA-native schemas under `docs/30-artifacts/schemas/`.

---

## 2) Non-negotiable rules

1. **Fail-closed on declared integrity:** if signatures/hashes are present and verification fails, do not bypass.
2. **No canon mutation under incident pressure:** canon changes require governed work; do not “hot-edit” Exchange.
3. **Prefer rollback over patching:** if activation/publish broke consumers, roll back to last-known-good and then fix forward.
4. **Preserve evidence:** do not overwrite logs/artifacts; snapshot first.
5. **Tenant isolation always:** no cross-tenant “quick fixes” (keys, packs, configs).

---

## 3) Roles and ownership

- **Incident Commander (IC):** directs response, owns timeline, decisions, and comms.
- **Operations Lead (Ops):** executes mitigations (rollbacks, holds, traffic shaping).
- **Security Lead (Sec):** keys, trust roots, revocations, suspected tampering.
- **Orgo Lead:** pipeline gates, build reproducibility, publish controls.
- **Konnaxion Lead:** activation/rollback, channel integrity, cache layout.
- **Architect Lead:** render determinism, trace coverage, template regressions.
- **SwarmCraft Lead:** execution failures, telemetry integrity, side-effect safety.

---

## 4) Severity model

- **SEV0:** active security compromise, cross-tenant breach, or widespread integrity bypass risk.
- **SEV1:** system-wide outage or incorrect canon distribution (bad pack broadly activated).
- **SEV2:** major degradation, partial outage, or correctness risk limited to a subset.
- **SEV3:** minor degradation, localized issue, workaround exists.
- **SEV4:** informational / no user impact.

Escalate to **SEV0/SEV1** when:
- signatures/hashes fail unexpectedly at scale,
- downgrade/substitution attempts are detected,
- trust roots/keys appear compromised,
- cross-tenant mixing is suspected,
- “no-new-facts” violations occur in regulated contexts.

---

## 5) Triage checklist (first 10 minutes)

### 5.1 Stabilize
- [ ] Assign IC + leads (Ops/Sec/Orgo/Konnaxion/Architect/SwarmCraft as needed).
- [ ] Freeze risky automation (publishes, auto-activation, key rotation) if it could worsen impact.
- [ ] Establish an incident channel + single source timeline.

### 5.2 Identify scope
- [ ] Which tenants/channels/environments are impacted?
- [ ] Which stage is failing: build, publish, activation, render, execute?
- [ ] Is this correctness, availability, or security?

### 5.3 Preserve evidence (before any destructive action)
Collect and snapshot:
- Orgo build record + stage logs (validation/compile/publish)
- Validation report(s) for failing builds
- Runtime Pack Manifest + channel index (as received)
- Activation record / local state from Konnaxion (active pack id, last-known-good, rejection reasons)
- Render bundles + trace maps for affected outputs
- SwarmCraft execution envelopes + telemetry for affected tasks
- Trust root set used for verification (per channel) + revocation state

---

## 6) Containment and mitigation playbooks

### 6.1 Bad release / bad pack activated
**Goal:** stop spread and restore last-known-good deterministically.

- [ ] Pause auto-activation for the affected channel(s).
- [ ] Roll back to last-known-good pack (per channel) using Konnaxion rollback procedure.
- [ ] Pin the channel to a known safe release (if your channel supports pinning).
- [ ] Quarantine the offending release (mark as revoked/blocked in distribution controls).
- [ ] Open a governed Orgo Case to fix forward.

### 6.2 Verification failures (signature/hash mismatch)
**Goal:** determine whether it is (a) packaging error, (b) wrong trust roots, (c) tampering.

- [ ] Confirm which verification step failed (index signature vs manifest signature vs file hash).
- [ ] Confirm the trust root set used for the channel is correct for the tenant/environment.
- [ ] Re-fetch artifacts via a trusted path (if network is involved) to rule out transient corruption.
- [ ] If mismatch persists:
  - treat as **SEV0/SEV1** until proven benign,
  - quarantine the artifact and suspect distribution integrity.
- [ ] If the trust root set is wrong:
  - fix by deploying corrected roots (tenant-scoped),
  - do not “accept both” unless explicitly governed and audited.

### 6.3 Schema validation failures (manifests / envelopes)
**Goal:** determine whether the producer emitted non-conformant artifacts or the consumer is pinned to the wrong version.

- [ ] Identify the schema version expected by the consumer.
- [ ] Confirm the producer’s pinned dependency versions (Kristal v4 commit/tag).
- [ ] If producer emitted non-conformant artifacts:
  - block publish (Orgo gate),
  - fix producer and regenerate artifacts.
- [ ] If consumer is pinned incorrectly:
  - roll back consumer deployment or adjust its pinned dependency,
  - do not relax validation in production as an emergency bypass.

### 6.4 Canon mismatch / reproducibility failure (same inputs, different IDs)
**Goal:** locate the source of non-determinism.

- [ ] Verify canonicalization settings are pinned and consistent across toolchains.
- [ ] Verify inputs are identical (input snapshot refs, blueprint/config refs, policy refs).
- [ ] Compare compilation environment metadata (compiler version, config hash, build constraints).
- [ ] If any dependency is not pinned, treat as root cause and remediate by pinning.
- [ ] Block publish until determinism is restored.

### 6.5 Rendering correctness incident (no-new-facts / trace failures)
**Goal:** prevent propagation of untraceable outputs.

- [ ] Disable or gate the affected templates/models (render pipeline) for impacted tenants.
- [ ] Switch to “safe rendering mode” (strict trace-required; omit if not traced).
- [ ] Capture render bundle + trace map for examples and failing assertions.
- [ ] If the issue is systematic:
  - roll back template/model bundle,
  - open a governed Orgo Case with reproduction steps and affected surfaces.

### 6.6 Execution incident (side effects, runaway tasks, unsafe actions)
**Goal:** stop harm, contain side effects, preserve audit.

- [ ] Halt task dispatch for affected task types/queues.
- [ ] Cancel or isolate running tasks if safe to do so (do not destroy evidence).
- [ ] Capture task envelopes, runtime logs, telemetry, and any external side-effect audit logs.
- [ ] Review mandate constraints (permissions, approvals, forbidden actions).
- [ ] Resume only after the control-plane gates are corrected.

### 6.7 Suspected security compromise (keys, trust roots, tampering, cross-tenant)
**Goal:** contain and rotate without breaking invariants.

- [ ] Treat as **SEV0**.
- [ ] Freeze publish/activation for impacted channels.
- [ ] Snapshot and lock down trust root stores, revocation sources, and signing infrastructure.
- [ ] Rotate keys per tenant/environment as required; publish revocations through the governed mechanism.
- [ ] Validate that no consumers accept the compromised keys post-rotation.
- [ ] Perform cross-tenant audit: ensure no shared roots were incorrectly configured.

---

## 7) Communication requirements

### 7.1 Internal updates (IC cadence)
- SEV0/SEV1: every 15 minutes
- SEV2: every 30 minutes
- SEV3/SEV4: as needed

Each update includes:
- what changed since last update,
- current impact/scope,
- mitigation status (rollback/pause/quarantine),
- next actions and owner.

### 7.2 External updates (if applicable)
Only publish externally when:
- scope is confirmed,
- a mitigation path exists,
- you can state clear customer impact and next steps.

Avoid speculation about root cause until verified.

---

## 8) Recovery and validation

Before declaring resolved:
- [ ] Impact has stopped (no new failures, no new incorrect outputs).
- [ ] Stable state achieved (last-known-good active; gates re-enabled safely).
- [ ] Verification checks pass end-to-end (publish → distribute → activate → render → execute).
- [ ] Monitoring confirms recovery (error rates, activation rejects, render trace failures, task failures).
- [ ] A governed Orgo Case exists for permanent fix (if not already created).

---

## 9) Post-incident requirements (within 24–72 hours)

### 9.1 Postmortem packet (required)
- Timeline (UTC)
- Impact scope (tenants/channels)
- Root cause analysis (technical + process)
- What worked / what didn’t
- Corrective actions (owners + deadlines)

### 9.2 Corrective actions (typical)
- Add/strengthen conformance tests (schemas, determinism checks, verification ordering).
- Improve guardrails (publish holds, canary activation, stricter rollback criteria).
- Update pinned dependencies and document them (Kristal v4 version pin).
- If contract changes are needed:
  - record a kOA ADR,
  - update integration profile docs,
  - never patch around contract divergence silently.

---

## 10) Quick reference: what to do first (by symptom)

- **Activation failing across a channel** → pause auto-activation → rollback → inspect verification step
- **New release causes crashes** → rollback → quarantine release → open fix-forward case
- **Signature/hash mismatch** → quarantine → verify trust roots → treat as security until proven otherwise
- **Schema validation errors** → block publish → confirm pinned versions → fix producer/consumer pinning
- **Untraceable rendered facts** → safe rendering mode → roll back template/model → collect bundles
- **Unsafe execution behavior** → halt dispatch → isolate tasks → capture telemetry → enforce mandate gates