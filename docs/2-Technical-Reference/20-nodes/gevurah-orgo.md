# Gevurah — Orgo (Governance + Control Plane)

## Purpose
**Orgo** is the ecosystem’s **governance and control plane**. It enforces the canonical stage spine (ingest → extract → resolve → validate → compile → distribute → render/execute), applies deterministic gates, and records an auditable trail of what happened and why.

Orgo is the owner of **kOA-native operational artifacts** (Cases, Tasks, Build Records, Release Records) and the authority that decides whether downstream steps are allowed to proceed.

---

## Responsibilities

### 1) Stage orchestration (normative)
Orgo MUST:
- Orchestrate the pipeline stages and enforce **stage order**.
- Provide idempotent execution semantics for each stage (retries must not create ambiguous state).
- Ensure each stage runs with explicit inputs, pinned configs, and recorded provenance.

### 2) Deterministic gating (normative)
Orgo MUST:
- Enforce **“no compile on fail”**: compilation and release actions are prohibited unless validation is passing.
- Enforce policy/blueprint selection as an explicit, recorded decision per build.
- Block downstream activation/release when integrity checks fail (**fail-closed**).

Kristal contract details are external; see:
- `docs/40-integration/kristal-v4/contract-pointers.md`
- `docs/40-integration/kristal-v4/conformance.md`

### 3) Governance workflow (cases/tasks) (normative)
Orgo MUST:
- Create, update, and close **Orgo Cases** and **Orgo Tasks**.
- Enforce task lifecycle rules (terminal states, reopen rules, ownership, priority/severity/visibility).
- Route work deterministically based on taxonomy and policy.

### 4) Reproducibility and audit (normative)
Orgo MUST:
- Emit a **Build Record** that references stage outputs and decisions.
- Emit a **Release Record** that ties a build to distribution intent (channels/cohorts/pins), and tracks rollout status.
- Store immutable audit logs for gate outcomes and changes in governance state.

### 5) Operator experience (recommended)
Orgo SHOULD:
- Provide a single operator view for: current build state, failures, validation outcomes, active releases, and rollback status.
- Provide tooling hooks to regenerate artifacts deterministically given the recorded inputs/configs.

---

## Inputs

### Upstream signals
- Ingest/provenance signals from Inputs (Chokmah)
- Blueprint/policy bundles from Mandate (Keter) and Blueprint (Binah)
- Resolution outputs from SenTient (Tiferet)
- Verification/activation telemetry from Konnaxion (Chesed)
- Execution telemetry (if present) from SwarmCraft

### Operator actions
- Create/triage/resolve cases and tasks
- Approve/pin/revoke releases
- Trigger rebuilds and rollbacks (within policy constraints)

---

## Outputs (Artifacts)

### kOA-native (owned by Orgo)
- `docs/30-artifacts/orgo-case.md`
- `docs/30-artifacts/orgo-task.md`
- `docs/30-artifacts/build-record.md`
- `docs/30-artifacts/release-record.md`

Schemas:
- `docs/30-artifacts/schemas/orgo-case.schema.json`
- `docs/30-artifacts/schemas/orgo-task.schema.json`
- `docs/30-artifacts/schemas/build-record.schema.json`
- `docs/30-artifacts/schemas/release-record.schema.json`

### Externally specified (referenced by Orgo; not defined here)
Orgo records and references Kristal artifacts (Exchange, Runtime Pack, Validation Report, etc.) but does not redefine them.
See `docs/40-integration/kristal-v4/contract-pointers.md`.

---

## Interfaces

### Control-plane API (recommended shape)
- Case/Task CRUD + lifecycle transitions
- Build orchestration endpoints (start/stop/retry stage, fetch build status)
- Release orchestration endpoints (create release intent, promote, pin, revoke, rollback)
- Read-only audit endpoints (gate decisions, stage timeline, artifact refs)

### Event stream (recommended)
Orgo SHOULD emit events for:
- Stage start/finish/fail
- Gate pass/fail (with stable reason codes)
- Case/Task lifecycle transitions
- Release intent changes and rollout milestones
- Rollback triggers and completion

### Storage (normative)
Orgo MUST persist:
- Case/Task state and history
- Build Records and Release Records
- Gate outcomes and operator actions (append-only or versioned history)

---

## Deterministic gates

Orgo’s gates are **policy-driven** but must be **deterministic** in outcome given the same inputs/configuration.

### Gate categories
1) **Schema/contract gates**
   - Validate kOA-native artifacts against kOA schemas.
   - Validate Kristal artifacts against pinned Kristal v4 schemas (external).

2) **Stage dependency gates**
   - A stage may only run if its dependencies are complete and valid.
   - Compilation/release prohibited unless validation passes.

3) **Integrity gates**
   - Downstream distribution/activation MUST be fail-closed.
   - Rollback MUST be deterministic given the same triggers and policy.

---

## Invariants (must always hold)

1) **No compile on fail**
- If validation fails, Orgo must not allow compilation, release intent, or activation.

2) **Explicit decisions are recorded**
- Policy/blueprint selection and any override MUST be recorded in the Build Record / Release Record.

3) **Auditability**
- Every terminal failure includes a stable reason code and traceable references to inputs and stage outputs.

4) **Idempotent stage execution**
- Retries do not create ambiguous dual outputs; Orgo records which output is authoritative for the build.

5) **Fail-closed rollout**
- Any verification/compatibility uncertainty blocks activation until resolved or explicitly overridden by policy (and recorded).

---

## Failure modes

### Pipeline failures
- Upstream input provenance missing or inconsistent
- Extract/resolve jobs produce invalid outputs
- Validation fails (must block compile/release)
- External dependency unavailable (artifact store, runner pool, key service)

### Governance failures
- Conflicting manual actions (double-approvals, invalid transitions)
- Partial updates causing inconsistent case/task state
- Release intent drift from actual rollout state

### Safety failures (critical)
- Attempted activation without verification
- Bypassing deterministic gates
- Unlogged administrative changes

---

## Observability

### Metrics (minimum)
- Build throughput, stage durations, queue depth
- Validation pass rate, top failure reasons
- Release success rate, time-to-rollout, rollback frequency
- Case/task lead time, backlog size, SLA compliance

### Logs (minimum)
- Gate decisions with reason codes and referenced artifact IDs
- Operator actions (who/what/when) with before/after state

### Traces (recommended)
- Correlate a build ID across all stage jobs and downstream distribution events
- Correlate a release ID to activation events, rollback events, and client health

---

## Security & access control

Orgo MUST enforce:
- Strong authentication for operator actions
- Authorization by role (operator, approver, auditor, automation)
- Immutable/auditable logging for privileged actions
- Secret isolation (keys used for signing/verifying are never exposed to untrusted jobs)

Orgo SHOULD:
- Support break-glass procedures with mandatory audit logging
- Separate duties for “approve release” vs “execute rollback” if required by policy

---

## Versioning & compatibility

- kOA-native artifact schemas evolve under `docs/90-reference/adr/`.
- Kristal artifact compatibility is governed by the pinned Kristal v4 dependency and the kOA profile:
  - `docs/40-integration/kristal-v4/pinned-dependency.md`
  - `docs/40-integration/kristal-v4/koa-profile.md`
  - `docs/40-integration/kristal-v4/legacy-compat.md`

---

## Links
- System gates: `docs/10-system/trust-boundaries.md`, `docs/10-system/determinism.md`
- Operations: `docs/50-operations/pipeline.md`, `docs/50-operations/releases.md`, `docs/50-operations/rollback.md`
- kOA-native artifacts: `docs/30-artifacts/`
- Kristal integration: `docs/40-integration/kristal-v4/`