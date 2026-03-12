# Rollbacks

**Purpose:** Restore a known-safe runtime state quickly and deterministically when a release causes correctness, availability, or safety risk.

Rollback is an operational action on **Runtime Pack activation state**. It does not modify canonical truth or Kristal artifact schemas.

---

## Non-negotiables

- **Fail-closed:** if a candidate pack cannot be verified or is incompatible, do not activate it.
- **Atomic activation:** activation is an all-or-nothing switch; partial activation is forbidden.
- **Downgrade prevention:** do not activate older versions unless explicitly authorized as a rollback.
- **Deterministic rollback:** given the same triggers and state history, rollback target selection must be deterministic.
- **Offline correctness:** rollback/activation must not depend on fetching trust roots over the network.

---

## Rollback modes

At minimum, support one (recommended: both):

1) **Pinned rollback**  
Activate an explicitly pinned known-good pack.

2) **Last-known-good (LKG) rollback**  
Activate the most recent previously active verified pack that is still present and passes preflight.

---

## When rollback is the correct action

Rollback is the correct action if:
- A newly activated pack causes elevated errors, regressions, or unsafe outcomes.
- Policy enforcement is not functioning as expected.
- Determinism or verification guarantees are in question.
- Activation is partially complete and must be normalized quickly.

Do not rollback (or rollback only after containment) if:
- The issue is purely upstream (inputs) and the active pack is not implicated.
- The issue is confined to a single target with environmental failure (prefer repair).
- The rollback candidate is known-bad for the same failure class.

---

## Triggers (examples)

Rollback can be triggered by:
- Explicit operator action (recommended primary trigger).
- Verified distribution control updates (e.g., revocation added, rollback authorization published).
- Local runtime health signals (optional; must be policy-defined).

---

## End-to-end shape (mental model)

```mermaid
flowchart LR
  A[Detect issue] --> B[Freeze / stop rollout]
  B --> C[Select rollback target (Pinned or LKG)]
  C --> D[Preflight: verify + compatibility]
  D -->|pass| E[Atomic activate rollback target]
  D -->|fail| F[Try earlier candidate or Safe Mode]
  E --> G[Post-check: health + correctness]
  G --> H[Record evidence + unfreeze (gated)]
````

---

## Procedure: automatic rollback (activation-time)

Automatic rollback is triggered when rollout detects:

* activation preflight failure
* post-activation health check failure
* policy enforcement failure (channel-defined)

Steps:

1. Stop further rollout in the same stage.
2. Attempt rollback to the **last-known-good** pack for the failing target(s).
3. If rollback succeeds:

   * mark rollout as failed
   * require human approval to proceed further
4. If rollback fails:

   * escalate to emergency procedure
   * place targets in **safe mode** if supported

---

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

* Enter **safe mode** (only if explicitly allowed for the channel):

  * deny/disable unsafe capabilities
  * serve minimal safe responses
  * disable optional modules suspected in failures

5. **Trust response (if integrity concern)**

* Rotate or revoke trust material as required by policy.
* Require conformance re-verification before any new promotion.

6. **Audit + incident**

* Open/attach an incident.
* Record all decisions and activation events in Release Record + Orgo.

---

## Rollback failure handling

If rollback activation fails on any target:

* Stop further actions for that stage.
* Attempt rollback to an earlier candidate (prior LKG) if policy allows.
* If no candidate passes preflight:

  * place target into safe mode (if supported)
  * escalate to incident response
  * require manual remediation

---

## Required observability and audit

During rollback, emit:

* per-target activation attempt events
* preflight outcomes (pass/fail + reason codes)
* post-activation health results
* final active pack ID per target

Dashboards/alerts should include:

* rollback success rate by channel and environment
* time-to-restore by incident
* repeated rollback loops (flapping)
* divergence: reported active pack ID vs expected

Audit linkage requirements:

* every rollback-related metric/event must reference `build_id` and/or `release_id` and relevant artifact refs (where applicable)

Konnaxion must also update and emit **Konnaxion State** on:

* rollback start and rollback completion
* failed verification or failed activation attempt

---

## Operational hygiene

* Maintain at least **N prior verified packs** per channel (policy-defined) to ensure rollback availability.
* Periodically perform **rollback drills** in non-prod and canary channels.
* Treat frequent rollbacks as a signal to investigate pipeline quality, conformance tests, and gate strictness.
* Ensure rollback tooling remains compatible across runtime versions (backward compatibility plan).

---

## Orgo integration

Operator-initiated and emergency rollbacks should be expressed as:

* an **Orgo Case** (incident/regression)
* one or more **Orgo Tasks**, such as:

  * `freeze-promotions`
  * `rollback-activation`
  * `post-rollback-validation`
  * `unfreeze-promotions` (optional, gated)

Rollback tasks must reference:

* channel/targets
* candidate selection rule or explicit candidate IDs
* required validations and success criteria


