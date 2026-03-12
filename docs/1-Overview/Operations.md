# Operations

This page describes how to run the system safely in production terms: how builds become releases, how rollouts are controlled, and how recovery works when something goes wrong.

## What “Operations” owns

Operations ensures that:
- Only **validated** outputs become candidates for release (“no compile on fail”).
- Only **verified** runtime packs activate (“fail closed”).
- Rollouts are controlled (channels/cohorts/pinning) and fully auditable.
- Rollbacks are deterministic and prevent partial activation.

## Golden path (build → release → runtime)

```mermaid
flowchart LR
  A[Build (Orgo)<br/>gated pipeline] --> B[Eligible output<br/>passed validation]
  B --> C[Release (channels/cohorts)<br/>promotion + monitoring]
  C --> D[Konnaxion<br/>verify -> activate]
  D --> E[Runtime (Malkuth)<br/>serve offline]
  E --> F[Rollback if needed<br/>deterministic target]
````

## Key operational concepts

* **Build**: a gated run that produces a candidate Runtime Pack and evidence of what happened.
* **Release**: controlled rollout of a Runtime Pack into an environment via channels/cohorts.
* **Channel**: a logical track (e.g., canary/stable/lts) with its own promotion and rollback behavior.
* **Cohort**: a controlled subset of traffic/tenants/devices for progressive rollout.
* **Pin**: holding a channel/cohort to a specific pack (freeze promotion).
* **Last-known-good (LKG)**: the deterministic fallback target used for rollback when not explicitly pinned.

## Build operations

### What happens in a build

* Inputs are ingested as immutable snapshots with provenance.
* Claim extraction/resolution happens upstream of truth.
* Validation is a hard gate.
* If validation passes, compilation produces canonical outputs + a Runtime Pack for distribution.

### What operators care about

* A build is **eligible** only if all gates pass.
* Every build produces operational evidence (records) so it can be audited and reproduced.

## Release operations

### Release lifecycle

1. **Create release** from an eligible build.
2. **Select channel + cohorts** for progressive rollout.
3. **Verify-before-activate**: integrity/compatibility checks must pass (fail closed).
4. **Monitor** health signals during rollout.
5. **Promote or pin** once confidence is sufficient.
6. **Record** outcomes and decisions (promotion/pin/rollback) in auditable release records.

### Operator checklist for a safe rollout

* Start with canary cohort(s)
* Require verification success before activation
* Promote in steps, not all at once
* Pin if signals are uncertain
* Roll back quickly if verification or health regresses

## Rollback operations

Rollback is the default recovery mechanism when a release is unsafe.

### Rollback triggers (examples)

* Pack verification failures
* Runtime health regressions during rollout
* Detected incompatibility with an environment/channel
* Policy violation or unexpected determinism drift

### Rollback invariants

* Target selection is deterministic (pinned target or LKG).
* Activation is atomic (avoid partial activation).
* Downgrade prevention policies may block unsafe rollback targets.
* Evidence is preserved (rollback is recorded; nothing is silently overwritten).

## Observability

Operations relies on two categories of signals:

### 1) System health signals (for go/no-go decisions)

* Verification pass/fail rates
* Activation success rates and latency
* Cohort/channel error rates and performance regressions
* Pack fetch/cache integrity signals (offline readiness)

### 2) Evidence and audit signals (for explainability)

* Build records: what inputs/policies were used and which gates passed/failed
* Release records: which pack was promoted/pinned/rolled back, where, and why

## Incident response (operator posture)

When an incident is declared:

1. **Stop the blast radius**: pause promotion and/or pin the channel.
2. **Prefer rollback** over patching in place.
3. **Preserve evidence** (records and traces) for root cause.
4. **Recover deterministically** (pinned/LKG targets; atomic activation).
5. **Document** what happened and what changed (post-incident record).


