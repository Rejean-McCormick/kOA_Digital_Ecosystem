# Releases

**Purpose:** Define how kOA turns a successful build into a controlled rollout, with verified distribution, activation, monitoring, and rollback.

This page describes **kOA release operations**. It does not define Kristal artifact formats.

## 1) Inputs to a release

A release is initiated when Orgo has:

* A **Build Record** (kOA-owned) for an eligible build
* A validation outcome of **PASS**
* References to Kristal-produced artifacts:

  * Exchange reference (opaque)
  * Runtime Pack reference (opaque)
* A pinned **Mandate Bundle** (governance + policy)
* A target rollout plan (channels/cohorts/regions)

## 2) Release stages (operational)

1. **Eligibility check**

   * Build status is PASS
   * Required approvals exist (per Mandate)
   * Required artifact references exist
   * Target channel is allowed

2. **Candidate publication**

   * Publish/announce the candidate Runtime Pack ref to the distribution store/channel tooling
   * Record the release intent (Release Record)

3. **Verification pre-check**

   * Confirm the candidate pack can be fetched
   * Confirm required signer keys are available and not revoked
   * Confirm compatibility policy allows this pack for the target cohort

4. **Rollout**

   * Gradual cohort expansion (canary → partial → full), or pinned rollout
   * Enforce “no silent jumps” (explicit cohort size changes)

5. **Monitoring**

   * Activation success rate
   * Verification failure reasons
   * Error budget / incident triggers

6. **Promotion / pin**

   * If metrics are healthy, promote to “latest” and/or pin
   * Record final state in Release Record

7. **Rollback (if triggered)**

   * Deterministic selection of rollback target (pinned or last-known-good)
   * Stop further rollout, revert cohorts, mark the candidate as rejected/revoked if needed

## 3) Channels, cohorts, and pinning

* **Channel:** logical lane (e.g., `canary`, `stable`, `lts`)
* **Cohort:** subset of clients (region, tenant, app version, feature flag)
* **Pinned:** clients must remain on a specific pack ID until explicitly changed

Rules (typical):

* Canary channel moves fastest, smallest cohorts
* Stable channel requires stricter thresholds
* LTS channel changes only with explicit approval and longer bake windows
* Pinning overrides “latest” resolution

## 4) Required invariants

* **Fail-closed activation:** clients must not activate an unverified pack.
* **Atomic switch:** activation is all-or-nothing.
* **Deterministic rollback:** rollback target selection must be deterministic under the policy.
* **No truth mutation:** releases change what pack is active, not canonical truth itself.
* **Auditable intent:** every promotion, pin, and rollback is recorded.

## 5) Rollback triggers (examples)

* Activation success rate drops below threshold
* Verification failures spike (e.g., signature mismatch, revoked signer)
* Crash/error rates exceed SLO
* Pack compatibility failures exceed threshold
* Security advisory or key compromise event

## 6) Operational records

kOA should record:

* **Release Record**: intent + current rollout state
* Activation metrics snapshots (by cohort/channel)
* Decisions: promote, pin, pause, rollback (with actor + reason)

## 7) What to reference for Kristal specifics

All field-level definitions for:

* Exchange / Exchange Manifest
* Runtime Pack / Runtime Pack Manifest
* Canonicalization and signing

are defined in the pinned Kristal v4 specification referenced in:

* `docs/40-integration/kristal-v4/contract-pointers.md`
