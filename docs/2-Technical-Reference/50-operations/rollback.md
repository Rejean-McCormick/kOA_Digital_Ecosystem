# Rollback

**Normative for kOA:** YES
**External normative reference:** Kristal v4 (pinned)

This runbook defines how kOA rolls back deployments safely and deterministically. Kristal v4 defines artifact validity; rollback procedures reference Kristal artifacts only by **opaque IDs** and rely on **verification** at activation time.

## Goals

* Restore a **last-known-good (LKG)** runtime state per channel/environment.
* Prevent partial or inconsistent activation across a rollout surface.
* Preserve auditability: every rollback produces a **Release Record** update (or a dedicated rollback record) and an Orgo trail.

## Definitions

* **Channel:** a named deployment lane (e.g., `prod`, `staging`, `canary`, `eu-prod`).
* **Target:** a concrete environment/cluster/edge group within a channel.
* **Active Pack:** the currently serving Runtime Pack for a target.
* **LKG Pack:** the most recent pack that was fully verified and healthy for the same target.
* **Rollback:** switching Active Pack from current → LKG (or another explicitly selected prior pack).

## Rollback types

1. **Automatic rollback (activation-time)**

   * Triggered when activation preflight fails or health checks fail during rollout.
2. **Operator-initiated rollback**

   * Triggered by incidents, regressions, policy/safety concerns, or customer impact.
3. **Emergency rollback**

   * Triggered by integrity compromise suspicion, unsafe behavior, or widespread outage.
   * May include freezes and trust/allowlist changes.

## Preconditions and safety gates

Rollback MUST be blocked if any of the following are true:

* No verified LKG pack exists for the target.
* The rollback candidate pack is not available in the artifact store.
* Activation preflight cannot run (e.g., verification subsystem down).
* The target is in a state requiring manual intervention (e.g., corrupted runtime, missing dependency) and “safe mode” must be used.

Rollback SHOULD be blocked (or require explicit override) if:

* The rollback candidate is older than policy permits for the channel.
* The rollback candidate is incompatible with the current runtime version.
* The rollback would violate an active governance constraint (unless emergency procedures apply).

## Inputs (required)

* `channel`
* `targets` (one or many)
* `rollback_reason` (structured: incident ID, regression ID, policy trigger)
* `rollback_mode` (automatic/operator/emergency)
* `candidate_pack_id` (optional; if omitted, use LKG)
* `release_record_id` (if rolling back a specific release)
* `operator_id` (for operator/emergency)

## Outputs (required)

* Updated **Release Record** (or a new rollback entry) including:

  * previous active pack ID
  * new active pack ID
  * scope (channel/targets)
  * timestamps
  * reason + authorizing entity
* Konnaxion activation events:

  * activation attempt IDs
  * per-target outcomes
* Orgo Tasks/Case updates (if operator initiated)

## Default policy

* Prefer **automatic rollback** on activation failure unless a channel explicitly disables it.
* Prefer **LKG rollback** unless the operator explicitly chooses another prior pack.
* Rollback is **idempotent**: repeating the same rollback request should not produce inconsistent state.

## Procedure: operator-initiated rollback (standard)

1. **Freeze promotions (channel)**

   * Prevent new activations while rollback runs.
   * If a release pipeline is mid-flight, halt at the next safe boundary.

2. **Select candidate**

   * If `candidate_pack_id` is not provided:

     * choose **LKG** for each target (can differ by target if policy allows)
   * If provided:

     * confirm it is a previously deployed pack for the same channel family (unless override)

3. **Preflight**

   * For each target:

     * verify candidate pack availability
     * run activation preflight (integrity + compatibility + policy gates)
     * confirm rollback will not strand dependent components (if runtime requires paired components)

4. **Execute rollback**

   * Perform activation to the candidate pack per target.
   * Use the channel’s rollout policy:

     * all-at-once (small surfaces)
     * staged (clusters → regions → global)
     * canary-first (recommended)
   * Enforce “no partial completion” unless explicitly configured:

     * if any stage fails, stop and proceed to “rollback of rollback” decision.

5. **Post-check**

   * Validate health and correctness signals:

     * runtime health checks
     * key business metrics
     * safety/policy guard telemetry
   * Confirm the active pack ID matches expected on all targets.

6. **Record and unfreeze**

   * Update Release Record / rollback entry.
   * Close/advance the Orgo Task.
   * Unfreeze promotions only after:

     * stability window criteria met (channel-defined), or
     * operator override with documented reason.

## Procedure: automatic rollback (activation-time)

Automatic rollback is triggered when rollout detects:

* activation preflight failure
* post-activation health check failure
* policy enforcement failure (channel-defined)

Steps:

1. Stop further rollout in the same stage.
2. Attempt rollback to the last-known-good pack for the failing target(s).
3. If rollback succeeds:

   * mark rollout as failed and require human approval to proceed.
4. If rollback fails:

   * escalate to emergency procedure.
   * place targets in safe mode if supported.

## Procedure: emergency rollback

Use when there is strong evidence of integrity compromise, unsafe behavior, or fast-spreading outage.

1. **Immediate freeze**

   * Freeze promotions and activations for the entire channel (or all channels if global).

2. **Scope containment**

   * Identify impacted targets and expand scope conservatively if uncertain.

3. **Rollback to LKG**

   * Execute rollback with priority on restoring safe operation.
   * If LKG is unavailable or fails preflight, select the most recent prior verified pack that passes preflight.

4. **If rollback cannot restore safety**

   * Enter **safe mode**:

     * deny/disable unsafe capabilities
     * serve minimal safe responses
     * disable optional modules that may be triggering failures
   * Keep service alive only if safe mode is explicitly approved for that channel.

5. **Trust response (if integrity concern)**

   * Rotate or revoke trust material as required by policy.
   * Require conformance re-verification before any new promotion.

6. **Audit + incident**

   * Open/attach an incident.
   * Record all decisions and activation events in Release Record + Orgo.

## Decision guide

Rollback is the correct action if:

* A newly activated pack causes elevated errors, regressions, or unsafe outcomes.
* Policy enforcement is not functioning as expected.
* Determinism or verification guarantees are in question.
* Activation is partially complete and must be normalized quickly.

Do not rollback (or rollback only after containment) if:

* The issue is purely upstream (inputs) and the active pack is not implicated.
* The issue is confined to a single target with environmental failure (prefer repair).
* The rollback candidate is known-bad for the same failure class.

## Rollback failure handling

If rollback activation fails on any target:

* Stop further actions for that stage.
* Attempt rollback to an earlier candidate (prior LKG) if policy allows.
* If no candidate passes preflight:

  * place target into safe mode (if supported)
  * escalate to incident response
  * require manual remediation

## Observability requirements

During rollback, the system must emit:

* per-target activation attempt events
* preflight outcomes (pass/fail + reason codes)
* post-activation health results
* the final active pack ID per target

Dashboards/alerts should include:

* rollback success rate by channel and environment
* time-to-restore by incident
* repeated rollback loops (flapping)
* divergence: reported active pack ID vs expected

## Operational hygiene

* Maintain at least **N prior verified packs** per channel (policy-defined) to ensure rollback availability.
* Periodically perform **rollback drills** in non-prod and canary channels.
* Treat frequent rollbacks as a signal:

  * investigate pipeline quality, conformance tests, and gating strictness.
* Ensure rollback tooling is compatible across runtime versions (backward compatibility plan).

## Orgo integration

Operator-initiated and emergency rollbacks must be expressed as:

* an Orgo Case (incident/regression)
* one or more Orgo Tasks:

  * `freeze-promotions`
  * `rollback-activation`
  * `post-rollback-validation`
  * `unfreeze-promotions` (optional gated task)

Rollback tasks must reference:

* channel/targets
* candidate selection rule or explicit candidate IDs
* required validations and success criteria
