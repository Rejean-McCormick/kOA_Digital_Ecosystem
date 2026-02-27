# 07 — Netzach: Architect-Strategy (Planning / Work Orchestration)

**File:** `docs/20-nodes/netzach-architect-strategy.md`  
**Status:** Normative (kOA)  
**External normative references:** Kristal v4 docs + schemas (pinned dependency)

---

## 1) Purpose

**Architect-Strategy** converts canonical truth + operating context into **governed work proposals**.
It does not render user-facing outputs and does not mutate canon. It produces **plans** that Orgo can accept, decompose into Cases/Tasks, execute, and audit.

---

## 2) Position in the system

### 2.1 Upstream inputs (typical)
- Kristal Exchange reference (pinned canonical truth)
- Mandate bundle (policy/constraints, approvals, prohibited actions)
- Blueprint (system configuration + interface constraints)
- Operational context (availability, incidents, release state, backlog, telemetry summaries)

### 2.2 Downstream consumers (typical)
- Orgo (for gating, decomposition, lifecycle management)
- SwarmCraft (execution of Orgo tasks; not directly commanded by Strategy)
- Observability/analytics (plan quality, drift, throughput)

---

## 3) Responsibilities (MUST / SHOULD)

### 3.1 MUST
- Produce **plan artifacts** that are schema-valid and auditable.
- Pin all dependencies used to construct a plan (exchange ref, mandate ref, blueprint ref, context snapshot ref).
- Separate **proposal** from **truth**:
  - Plans may *recommend* changes/work, but do not assert canon updates.
- Emit work in **governable units**:
  - proposals must map cleanly to Orgo Cases/Tasks with explicit owners, types, priorities, and constraints.
- Respect mandate constraints (permissions, risk thresholds, forbidden actions, review requirements).
- Provide explicit **rationale** and **evidence pointers** for each proposed work item.

### 3.2 SHOULD
- Produce plans that are stable under small input changes (reduce churn).
- Prefer incremental diffs to prior plans when possible (reduce operational noise).
- Detect conflicts (duplicate work, incompatible tasks, dependency cycles) and surface them explicitly.
- Provide estimated impact/cost/risk fields suitable for Orgo routing and approval.

---

## 4) Inputs (interfaces)

Architect-Strategy consumes references, not raw truth bytes, unless explicitly allowed by policy.

### 4.1 Required references (recommended baseline)
- `exchange_ref` (points to the Kristal Exchange used)
- `mandate_ref` (policy bundle / mandate version)
- `blueprint_ref` (system contract/config bundle)
- `context_ref` (serialized summary of operational context and telemetry inputs)

### 4.2 Optional inputs
- Previous plan reference (for diffing / continuity)
- Open Orgo cases/tasks snapshot (for dedupe / routing)
- Release/channel state (for rollout-aware planning)

---

## 5) Outputs (artifacts)

Architect-Strategy produces **pre-truth** artifacts only.

### 5.1 Primary output: Plan Envelope (kOA artifact)
A plan envelope is a typed object that includes:
- pinned dependency references (exchange/mandate/blueprint/context)
- proposed cases/tasks (or pointers to them)
- ordering/precedence constraints and dependency edges
- gating requirements (human review, multi-party approval, risk signoff)
- rationale and evidence pointers

### 5.2 Secondary outputs (optional)
- Plan diff vs previous plan
- Risk register entries
- Coverage summary (what domains/areas were assessed, what was intentionally omitted)

---

## 6) Invariants (non-negotiable)

1. **No canon mutation:** Strategy cannot write or patch Kristal Exchange.
2. **Pinned inputs:** Every plan must name the exact Exchange + policy/config + context it was derived from.
3. **Governed work only:** All actions must be representable as Orgo Cases/Tasks (no direct execution commands).
4. **Policy compliance:** A plan that violates mandate constraints is invalid.
5. **Auditability:** Every proposed work item has rationale + evidence pointers sufficient for review.

---

## 7) Determinism and reproducibility

Strategy may use probabilistic methods, but it must be **auditable and reproducible enough for governance**.

### 7.1 Required
- Record generator identity/version and key parameters.
- Record a stable `plan_id` and `plan_schema_version`.
- Record a `context_ref` that is sufficient to explain the plan.

### 7.2 Recommended
- Record a `random_seed` (or equivalent) when stochastic behavior is used.
- Provide a “replay mode” that can regenerate the same plan given the same pinned inputs.

---

## 8) Failure modes

### 8.1 Input failures
- Missing or incompatible `exchange_ref` / `mandate_ref` / `blueprint_ref`
- Stale or invalid context snapshot
- Insufficient permissions (mandate prohibits required actions)

### 8.2 Planning failures
- Dependency cycles in proposed tasks
- Duplicate/conflicting work proposals
- Excessive churn (plan oscillation without meaningful input change)

### 8.3 Output failures
- Plan envelope schema invalid
- Missing rationale/evidence pointers
- Tasks not representable in Orgo taxonomy (unknown type/category/owner)

---

## 9) Orgo interaction model

Architect-Strategy proposes; Orgo decides and enforces.

### 9.1 Submission
- Strategy submits Plan Envelope to Orgo intake.
- Orgo validates:
  - schema validity
  - mandate compliance
  - compatibility with current release/channel state
  - dedupe vs open work

### 9.2 Decomposition
- Orgo decomposes plan into Cases/Tasks (or accepts Strategy-provided tasks if allowed).
- Orgo assigns routing, approvals, and execution constraints.

### 9.3 Feedback loop
- Orgo returns acceptance/rejection with reasons.
- Strategy may revise plans, but must preserve audit trail (superseding plan links).

---

## 10) Observability

### 10.1 Logs/events (minimum)
- plan_generated (with plan_id, pinned refs, counts by task type, churn metrics)
- plan_rejected (reason codes, violated constraints)
- plan_accepted (case/task counts, routing summary)

### 10.2 Metrics (recommended)
- plan_generation_latency
- plan_acceptance_rate / rejection_rate
- task_dedupe_rate
- plan_churn_rate (diff size over time)
- policy_violation_attempt_rate
- downstream_execution_outcomes (success/failure distribution by plan lineage)

---

## 11) Security and access control

- Strategy must operate under least-privilege:
  - read access to pinned inputs it is authorized to consume
  - write access only to pre-truth proposal artifacts
- Strategy must not access tenant data outside scope of the current mandate.
- Any cross-tenant/shared planning is an explicit design choice and must be governed.

---