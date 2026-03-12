# Operations — Releases

Releases are how validated knowledge moves into runtime safely. A release takes an eligible build output (Runtime Pack) and rolls it out through controlled stages (channels/cohorts), with verification gates, monitoring, and reversible activation.

## Goals

- Deliver new knowledge to users safely and predictably
- Prevent partial or unsafe activations (fail-closed)
- Support controlled rollout (canary → stable) and fast rollback
- Preserve evidence: what was released, where, why, and with which results

## Core concepts

### Build vs Release
- **Build**: the result of the pipeline producing artifacts (including a Runtime Pack) that are eligible for distribution.
- **Release**: the operational act of distributing and activating a specific Runtime Pack in one or more environments/channels.

### Channel
A named release lane (examples: `canary`, `stable`, `lts`). Channels define default rollout speed, guardrails, and who receives updates.

### Cohort
A subset within a channel (by tenant group, region, percent rollout, etc.). Cohorts let you ramp exposure gradually.

### Pinning
Holding a channel (or cohort) to a specific pack/version to stop automatic upgrades until intentionally changed.

### Last-known-good (LKG)
A known safe pack/version you can roll back to deterministically.

## Release lifecycle (high level)

1. **Request**
   - A release is requested for a specific build output (Runtime Pack reference) and a target (env + channel + optional cohort plan).

2. **Prepare**
   - Resolve targets (channels/cohorts), confirm eligibility, and collect the operational record that will track the release.

3. **Distribute**
   - Make the pack available to runtime distribution (fetchable by Konnaxion).
   - Distribution must not imply activation.

4. **Verify (fail-closed)**
   - Integrity, compatibility, and policy checks must pass before activation is allowed.

5. **Activate (atomic)**
   - Activation is an atomic switch of “active pack” for the target scope.
   - No partial activation is allowed.

6. **Monitor**
   - Observe key health signals over a defined window (errors, latency, correctness signals, downstream impacts).

7. **Promote / Expand**
   - Expand cohorts (or promote from canary → stable) only after monitoring criteria are met.

8. **Pin / Hold (optional)**
   - Pin a channel/cohort to lock it to a specific pack/version.

9. **Close**
   - Record outcomes (success, rolled back, pinned, aborted) and link postmortems if needed.

## Eligibility (what can be released)

A pack is eligible only if:
- It was produced from a validated pipeline run (validation gate passed).
- It matches the target’s compatibility constraints (schema/profile/runtime).
- It has all required references for audit (provenance and build identity).

## Verification gates (non-negotiable)

Release must fail closed if any of the following fails:
- Pack integrity (hash/signature/manifest mismatch)
- Schema/profile compatibility with the pinned runtime expectations
- Policy blocks (forbidden downgrade, forbidden source, forbidden scope, etc.)
- Missing required evidence records (build identity, provenance references)

## Operator workflow

### Pre-flight checklist
- Confirm target environment(s) and channels
- Confirm cohort plan (if used) and monitoring window
- Confirm rollback target strategy (LKG or pinned prior)
- Confirm alerting/observability is in place for the release window
- Confirm the release has an owner and an escalation path

### Execution checklist
- Start release in smallest safe scope (canary / smallest cohort)
- Confirm verification gates passed before any activation
- Activate atomically
- Monitor until criteria are met or violated
- If clean: expand cohort / promote channel
- If not clean: roll back deterministically and preserve evidence

## Monitoring (what “good” looks like)

Define success criteria per release. Typical signals:
- Activation success rate and time-to-activate
- Runtime error rates (spikes, new error classes)
- Latency regressions (p95/p99)
- Data correctness signals (trace coverage, query mismatch alarms)
- Tenant-impact indicators (support tickets, escalation rate)

Minimum monitoring windows:
- Canary: short but real (enough to catch immediate failures)
- Stable: longer (enough to catch delayed regressions)

## Promotion and expansion

Promotion is gated by:
- Canary health within thresholds
- No new critical errors introduced
- No policy violations observed
- Rollback readiness confirmed (LKG available and valid)

Recommended strategy:
- **Canary first**, then expand cohorts gradually, then promote to stable.

## Pinning strategy (when to pin)

Pin when you need:
- Controlled freeze during incident response
- A stable baseline for audits or regulated workflows
- A compatibility hold while upstream/downstream dependencies catch up

Pinning should be time-bounded and recorded with rationale.

## Rollback (what triggers it)

Rollback triggers:
- Verification failure (never activate)
- Activation failure (immediate rollback)
- Monitoring failure (regression detected)
- Policy violation discovered post-activation
- Unexpected tenant impact

Rollback principles:
- Deterministic target selection (pinned or LKG)
- Atomic activation of the rollback target
- Preserve the release record and link incident notes

See: [Operations — Rollbacks](Operations-Rollbacks)

## Evidence and audit trail

Every release should leave an auditable trail:
- Release identity (who/what/when)
- Target scope (env/channel/cohort)
- Pack identity (what was activated)
- Gate outcomes (verification results, activation result)
- Monitoring window + results
- Promotion/pinning/rollback decisions + rationale
- Links to incident/postmortem if applicable

See: [Artifacts — Operational](Artifacts-Operational)

## Common pitfalls

- Activating without a verified pack (must be fail-closed)
- Rolling out too broadly before monitoring criteria are met
- Not defining a rollback plan before activating
- Allowing “floating latest” dependencies that break determinism
- Missing evidence links (hard to debug, hard to audit)

## Related pages

- [Operations](Operations)
- [Operations — Builds](Operations-Builds)
- [Operations — Rollbacks](Operations-Rollbacks)
- [Operations — Observability](Operations-Observability)
- [Operations — Incident response](Operations-Incident-response)
- [Integration — Kristal v4](Integration-Kristal-v4)