# Integration — External systems

This page describes **how kOA connects to systems outside the ecosystem** without breaking the core invariants (truth boundary, determinism, fail-closed gates, offline correctness).

kOA integrates with external systems through **explicit adapters and artifacts**—not implicit shared state.

---

## Integration map (conceptual)

```mermaid
flowchart LR
  subgraph External["External systems"]
    S1[Content sources\n(docs, feeds, uploads)]
    S2[Systems of record\n(services, databases)]
    S3[Automation\n(schedule/webhook/CI)]
    S4[Identity & secrets\n(SSO, KMS/HSM, secret store)]
    S5[Observability\n(logs/metrics/traces)]
    S6[Runtime consumers\n(apps/services/devices)]
  end

  S1 --> A[Ingest adapters\n-> snapshots + provenance]
  S2 --> A
  S3 --> O[Orgo\ncontrol plane]
  S4 --> O
  S4 --> K[Konnaxion\nverify/activate/rollback]
  S5 <--> O
  S5 <--> K
  A --> P[Pipeline stages\n(extract/resolve/validate/compile)]
  P --> K --> S6
````

---

## 1) Ingest sources (how external inputs enter safely)

**Goal:** turn external inputs into **immutable snapshots with provenance**, so builds are replayable and auditable.

Common source types:

* Documents and file drops (batches)
* Feeds and APIs (polled or pushed)
* Operational inputs (tickets, incidents, runbooks)
* Manual uploads (operator-triggered)

Integration pattern:

* Build an **ingest adapter** per source type.
* The adapter produces **snapshots** (content-addressed is recommended) plus **provenance pointers** (opaque references are fine).
* Credentials and access policies are managed outside the adapter logic (see Security).

Design rules:

* Do not let “live” external state flow downstream un-snapshotted.
* Treat ingest as **capture + provenance**, not interpretation.

---

## 2) Automation triggers (how work starts)

External systems may initiate work via:

* **Manual** operator requests
* **Schedules** (cron/CI timers)
* **Webhooks** (push events from external systems)
* **Incident** triggers (operational events)
* **Release** triggers (promotion/rollback workflows)

Integration pattern:

* External triggers create or update **governed work** in Orgo (Cases/Tasks).
* Triggers must be recorded with “who/why/where from” metadata (for auditability).

Rule:

* A trigger may start work, but it **cannot bypass gates** (validation, verification-before-activation, etc.).

---

## 3) Runtime consumers (how downstream products use knowledge)

**Goal:** downstream consumers receive **verified Runtime Packs** and can operate **offline**.

Typical consumers:

* Edge nodes / devices
* Internal services that need fast local query
* User-facing applications that must remain deterministic and traceable

Integration pattern:

* Consumers talk to Konnaxion (or the platform layer it powers) to obtain packs.
* Activation is **atomic** and can be **rolled back deterministically**.
* Consumers should treat the pack as their source of facts; avoid coupling correctness to live external calls.

Rule:

* If a consumer needs “freshness,” it is handled by **controlled pack rollout**, not by bypassing the pack.

---

## 4) Security integrations (identity, secrets, key management)

**Goal:** external security platforms provide identity and cryptographic trust while kOA enforces fail-closed behavior.

What typically integrates here:

* **SSO / identity provider** (human access)
* **Workload identity** (service-to-service auth)
* **Secret manager** (adapter credentials, CI secrets, non-prod keys)
* **KMS/HSM** (distribution signing keys, trust roots, rotation)

Integration rules:

* Production verification must rely on **production-trusted keys only** (no dev/test keys).
* Key rotation must be operationally safe (planned and auditable).
* Authorization must respect environment and tenant boundaries.

---

## 5) Observability and audit integrations

**Goal:** external observability stacks make the system operable, and audit evidence stays linkable.

What to emit (minimum):

* Build and release identifiers
* Artifact references (snapshot-set, validation evidence, pack identifiers)
* Gate outcomes (validation pass/fail, verify pass/fail, activation/rollback)
* Deterministic refusal/error codes for downstream stages

Integration pattern:

* Logs/metrics/traces go to your existing stack (SIEM, metrics store, tracing backend).
* Audit evidence should be recoverable by operators and correlate to releases and rollbacks.

Rule:

* “What happened?” must be answerable from records without re-running the pipeline.

---

## 6) Multi-environment and multi-tenant considerations

Recommended separation:

* Separate channels/environments (dev/staging/prod) with distinct trust roots and signing policies.
* Explicit tenant isolation for snapshots, packs, and activation state.
* Prevent cross-environment artifact reuse unless explicitly allowed by policy.

Rule:

* Treat boundaries as security boundaries: keys, identities, and activation state should not bleed across.

---

## Integration checklist (non-technical)

Before connecting an external system, confirm:

* [ ] The integration point is an **adapter or artifact boundary**, not shared mutable state.
* [ ] Inputs become **snapshots + provenance** before entering the pipeline.
* [ ] Triggers create **governed work** and do not bypass gates.
* [ ] Runtime consumers rely on **verified packs** (offline correctness preserved).
* [ ] Keys/secrets are managed by **approved identity and key platforms**.
* [ ] Observability ties events to **build/release ids** and gate outcomes.
* [ ] Rollback story is clear (what happens on verify failure or bad rollout).

---

## Links

* [Integration (Overview)](Integration)
* [Integration — Kristal v4](Integration-Kristal-v4)
* [Operations](Operations)
* [Artifacts](Artifacts)

