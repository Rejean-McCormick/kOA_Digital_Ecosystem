# Operations — Incident response

**Status:** Normative (kOA)  
**External normative references:** Kristal v4 docs + schemas (pinned dependency)

---

## 1) Purpose

Provide a deterministic, auditable procedure for detecting, triaging, mitigating, and resolving incidents across:
- Orgo build pipeline (validation/compile/publish)
- Konnaxion distribution/activation/rollback
- Architect rendering (trace / no-new-facts)
- SwarmCraft execution (side effects + telemetry)
- Trust roots / keys / revocations
- Multi-tenant isolation

This runbook avoids schema-level duplication. When you need artifact formats, follow the pinned Kristal references or kOA-native schemas.  

---

## 2) Non-negotiable rules

1. **Fail-closed on declared integrity** (no bypass on signature/hash failures).
2. **No canon mutation under incident pressure** (no “hot edits” to canon).
3. **Prefer rollback over patching** when a publish/activation broke consumers.
4. **Preserve evidence** (snapshot first; do not overwrite).
5. **Tenant isolation always** (no cross-tenant “quick fixes”).

---

## 3) Roles and ownership

- **Incident Commander (IC):** timeline, decisions, comms, final resolution call.
- **Operations Lead (Ops):** freezes/holds, rollbacks, rollout controls.
- **Security Lead (Sec):** keys, trust roots, revocations, tampering response.
- **Orgo Lead:** pipeline gates, reproducibility, publish controls.
- **Konnaxion Lead:** activation/rollback, channel integrity, cache/runtime state.
- **Architect Lead:** render determinism, trace coverage, template/model regressions.
- **SwarmCraft Lead:** execution failures, telemetry integrity, side-effect containment.

---

## 4) Severity model

- **SEV0:** active compromise, cross-tenant breach, or integrity-bypass risk.
- **SEV1:** system-wide outage or incorrect canon distribution (bad pack broadly activated).
- **SEV2:** major degradation or correctness risk limited to a subset.
- **SEV3:** minor degradation, localized issue, workaround exists.
- **SEV4:** informational / no user impact.

Escalate to **SEV0/SEV1** when integrity or isolation is uncertain (signature/hash failures at scale, downgrade/substitution attempts, key compromise suspicion, cross-tenant mixing, or regulated “no-new-facts” violations).

---

## 5) Triage checklist (first 10 minutes)

### 5.1 Stabilize
- [ ] Assign IC + required leads.
- [ ] Freeze risky automation if it could worsen impact (publishes, auto-activation, key rotation).
- [ ] Establish incident channel + single source of timeline truth.

### 5.2 Identify scope
- [ ] Which tenants/channels/environments are impacted?
- [ ] Which stage is failing: build, publish, activation, render, execute?
- [ ] Is this correctness, availability, or security?

### 5.3 Preserve evidence (before destructive actions)
Snapshot:
- [ ] Orgo Build Record + stage logs
- [ ] Validation report(s) for failing builds
- [ ] Runtime Pack Manifest + channel index (as received)
- [ ] Konnaxion activation/local state (active pack, LKG, rejection reasons)
- [ ] Render bundles + trace maps for affected outputs
- [ ] SwarmCraft execution envelopes + telemetry for affected tasks
- [ ] Trust root set (per channel) + revocation state

---

## 6) Containment and mitigation playbooks

### 6.1 Bad release / bad pack activated
**Goal:** stop spread and restore last-known-good deterministically.
- [ ] Pause auto-activation for affected channel(s).
- [ ] Roll back to last-known-good pack (per channel/targets).
- [ ] Pin channel to a known safe release (if supported).
- [ ] Quarantine offending release (revoke/block in distribution controls).
- [ ] Open a governed Orgo Case to fix forward.

### 6.2 Verification failures (signature/hash mismatch)
**Goal:** decide if this is packaging error, wrong trust roots, or tampering.
- [ ] Identify which verification step failed (index vs manifest vs payload hash).
- [ ] Confirm tenant/channel trust roots are correct.
- [ ] Re-fetch artifacts via a trusted path to rule out transient corruption.
- [ ] If mismatch persists: treat as **SEV0/SEV1** until proven benign; quarantine artifacts.
- [ ] If trust roots are wrong: deploy corrected, tenant-scoped roots; do not broaden acceptance without governance + audit.

### 6.3 Schema validation failures (manifests / envelopes)
**Goal:** detect non-conformant producer vs incorrect consumer pinning.
- [ ] Identify schema version expected by the consumer.
- [ ] Confirm producer pinned dependency version (Kristal v4 tag/commit).
- [ ] If producer is non-conformant: block publish at Orgo gate; fix producer; regenerate.
- [ ] If consumer pinning is wrong: roll back consumer deployment or correct pinning; do not relax validation in production.

### 6.4 Canon mismatch / reproducibility failure (same inputs, different IDs)
**Goal:** locate non-determinism source.
- [ ] Confirm canonicalization settings are pinned and consistent.
- [ ] Confirm inputs are identical (snapshot refs, blueprint/config refs, policy refs).
- [ ] Compare toolchain/environment metadata.
- [ ] If any dependency is unpinned: treat as root cause; remediate by pinning.
- [ ] Block publish until determinism is restored.

### 6.5 Rendering correctness incident (no-new-facts / trace failures)
**Goal:** prevent propagation of untraceable or policy-violating outputs.
- [ ] Disable/gate affected templates/models for impacted tenants.
- [ ] Switch to safe rendering mode (strict trace-required; omit/refuse when not traced).
- [ ] Capture render bundle + trace map examples and failing assertions.
- [ ] If systematic: roll back template/model bundle; open a governed Orgo Case with repro steps.

### 6.6 Execution incident (side effects, runaway tasks, unsafe actions)
**Goal:** stop harm, contain side effects, preserve audit.
- [ ] Halt dispatch for affected task types/queues.
- [ ] Isolate/cancel running tasks when safe (do not destroy evidence).
- [ ] Capture task envelopes, runtime logs, telemetry, and external side-effect audit logs.
- [ ] Review mandate constraints (permissions, approvals, forbidden actions).
- [ ] Resume only after control-plane gates are corrected.

### 6.7 Suspected security compromise (keys, trust roots, tampering, cross-tenant)
**Goal:** contain and rotate without breaking invariants.
- [ ] Treat as **SEV0**.
- [ ] Freeze publish/activation for impacted channels.
- [ ] Snapshot and lock down trust root stores, revocation sources, and signing infrastructure.
- [ ] Rotate keys per tenant/environment; publish revocations via governed mechanism.
- [ ] Validate that consumers reject compromised keys post-rotation.
- [ ] Perform cross-tenant audit (ensure no shared roots were incorrectly configured).

---

## 7) Communication requirements

### 7.1 Internal updates (IC cadence)
- SEV0/SEV1: every 15 minutes
- SEV2: every 30 minutes
- SEV3/SEV4: as needed

Each update includes:
- what changed since last update
- current impact/scope
- mitigation status (freeze/rollback/quarantine)
- next actions + owner

### 7.2 External updates (if applicable)
Only publish externally when:
- scope is confirmed
- mitigation path exists
- you can state clear customer impact and next steps

Avoid speculation about root cause until verified.

---

## 8) Recovery and validation

Before declaring resolved:
- [ ] Impact has stopped (no new failures / incorrect outputs).
- [ ] Stable state achieved (LKG active; gates re-enabled safely).
- [ ] Verification checks pass end-to-end (publish → distribute → activate → render → execute).
- [ ] Monitoring confirms recovery (errors, verify rejects, trace failures, task failures).
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
- Strengthen conformance tests (schema, determinism, verification ordering).
- Improve guardrails (publish holds, canary activation, stricter rollback criteria).
- Update pinned dependencies and document them.
- If contract changes are needed: record a kOA ADR; update integration profile docs; never patch around divergence silently.

---

## 10) Quick reference (by symptom)

- **Activation failing across a channel** → pause auto-activation → rollback → inspect verification step
- **New release causes crashes** → rollback → quarantine release → open fix-forward case
- **Signature/hash mismatch** → quarantine → verify trust roots → treat as security until proven otherwise
- **Schema validation errors** → block publish → confirm pinned versions → fix producer/consumer pinning
- **Untraceable rendered facts** → safe rendering mode → roll back template/model → collect bundles
- **Unsafe execution behavior** → halt dispatch → isolate tasks → capture telemetry → enforce mandate gates