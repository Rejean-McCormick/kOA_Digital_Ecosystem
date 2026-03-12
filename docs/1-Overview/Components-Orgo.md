# Orgo (Control Plane)

**Role:** Governance + pipeline control plane. 
Orgo enforces the canonical stage spine (ingest → extract → resolve → validate → compile → distribute → render/execute), applies deterministic gates, and records an auditable trail of what happened and why. 

```mermaid
flowchart LR
  O[Orgo<br/>control plane] --> S[Stage spine<br/>orchestration + gates]
  S --> BR[Build Record]
  S --> RR[Release Record]
  S --> D[Distribute/Activate<br/>(Konnaxion)]
  S --> R[Render/Execute<br/>(Architect/SwarmCraft)]
```

## What Orgo owns (and what it doesn’t)

### Owns

* **Stage orchestration:** enforce stage order; run stages with explicit inputs and pinned configs; idempotent retries. 
* **Deterministic gating:** enforce “no compile on fail”; block downstream activation/release when integrity checks fail (fail-closed). 
* **Governance workflow:** Cases/Tasks lifecycle and deterministic routing. 
* **Audit + reproducibility evidence:** Build Record + Release Record; immutable audit logs for gate outcomes and governance changes. 

### Does not own

* Kristal artifact formats/schemas/canonicalization rules (treated as external, pinned contracts). 

## Responsibilities (operator-facing)

### 1) Enforce the stage spine

Orgo enforces an ordered pipeline from ingest through activation and feedback. 

### 2) Make “truth gating” non-negotiable

If validation fails, Orgo stops the pipeline and must not run compile (“no compile on fail”). 

### 3) Make releases controlled and auditable

Orgo ties an eligible build to a rollout intent (channels/cohorts/pins), monitors rollout, and drives rollback when needed—while recording the full decision trail in Release Records. 

## Inputs (what Orgo listens to)

### Upstream signals

* Ingest/provenance signals (Chokmah) 
* Blueprint/policy bundles (Keter/Binah) 
* Resolution outputs (SenTient) 
* Verification/activation telemetry (Konnaxion) 
* Execution telemetry (SwarmCraft, if present) 

### Operator actions

* Create/triage/resolve Cases and Tasks; approve/pin/revoke releases; trigger rebuilds/rollbacks (within policy). 

## Outputs (what Orgo produces)

### kOA-native artifacts (owned by Orgo)

* Orgo Case, Orgo Task, Build Record, Release Record. 

Orgo persists a **Build Record** for every pipeline execution and a **Release Record** for every promotion/publish action. 

## Deterministic gates (what Orgo blocks/permits)

Orgo gates are policy-driven but must be deterministic for the same inputs/configuration. 

Gate categories:

1. **Schema/contract gates:** validate kOA-native artifacts against kOA schemas; validate Kristal artifacts against pinned Kristal v4 schemas. 
2. **Stage dependency gates:** stages run only when dependencies are complete/valid; compile/release prohibited unless validation passes. 
3. **Integrity gates:** distribution/activation must be fail-closed; rollback must be deterministic under policy. 

## Invariants (must always hold)

* **No compile on fail:** validation failure blocks compilation, release intent, and activation. 
* **Explicit decisions are recorded:** policy/blueprint selection and overrides are recorded in Build/Release Records. 
* **Auditability:** terminal failures include stable reason codes and traceable references. 
* **Idempotent stage execution:** retries do not create ambiguous dual outputs; Orgo records what is authoritative. 
* **Fail-closed rollout:** verification/compat uncertainty blocks activation unless policy explicitly overrides (and it’s recorded). 

## Interfaces (recommended shapes)

### Control-plane API

* Case/Task CRUD + lifecycle transitions
* Build orchestration (start/stop/retry stage, fetch build status)
* Release orchestration (create intent, promote, pin, revoke, rollback)
* Read-only audit endpoints (gate decisions, stage timeline, artifact refs) 

### Event stream

Orgo should emit events for stage start/finish/fail, gate pass/fail (with stable reason codes), Case/Task transitions, rollout milestones, rollback triggers/completion. 

### Storage (non-negotiable)

Orgo persists Case/Task history, Build/Release Records, gate outcomes, and operator actions with append-only or versioned history. 

## Failure modes (classes)

* Pipeline failures: missing/incorrect provenance; invalid extractor/resolver outputs; validation failures (must block compile/release). 
* Governance failures: conflicting manual actions; inconsistent case/task state; intent drift vs rollout state. 
* Safety failures (critical): attempted activation without verification; bypassed gates; unlogged admin changes. 

## Observability (minimum)

Metrics and logs should make it easy to answer:

* What build/release is failing, where, and why?
* Which reason codes are trending?
* How often are rollbacks happening, and what triggered them?

Minimum metrics: build throughput/durations, validation pass rate and top reasons, release success/time-to-rollout, rollback frequency, case/task lead time/backlog. 
Minimum logs: gate decisions with reason codes and referenced artifact IDs; operator actions with before/after state. 
Recommended traces: correlate build IDs across stage jobs and downstream distribution; correlate release IDs to activation/rollback and client health. 

## Related pages

* Pipeline operations (Orgo): `Operations-Builds` / `Operations` 
* Orgo-native artifacts: Build Record / Release Record / Case / Task 
