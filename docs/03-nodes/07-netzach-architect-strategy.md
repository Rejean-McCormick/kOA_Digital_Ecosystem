# Netzach (Strategy / Direction) — Architect-Strategy

Architect-Strategy is the ecosystem’s planning and direction node. It converts **mandates + canonical truth (Kristal Exchange) + runtime capabilities** into **governed Plans** that Orgo can express as **Cases/Tasks**, and SwarmCraft can execute.

Architect-Strategy produces **proposals for action**, not truth. Canonical truth is owned by Kristal; execution is owned by SwarmCraft; enforcement of gates is owned by Orgo.

---

## 1) Scope and non-goals

### In scope
- Convert objectives into an actionable plan: decomposition, sequencing, dependencies, risk controls.
- Translate plans into Orgo-native work structures (Cases/Tasks, routing labels, priority, policies).
- Select and configure execution approach (tools, agents, human steps) within the Blueprint and mandate constraints.
- Maintain an auditable plan record: assumptions, constraints, and justification pointers to canon.

### Out of scope
- Canon creation or mutation (Kristal Exchange is authoritative).
- Rendering user-facing content (owned by Architect-Render).
- Direct execution of steps (owned by SwarmCraft).
- Overriding gates and policy enforcement (owned by Orgo).

---

## 2) Responsibilities

## 2.1 Planning (goal → plan)
- Interpret the mandate into explicit goals, constraints, success criteria, and acceptance tests.
- Identify required canonical inputs (which Kristal IDs / pack versions / data projections).
- Create a plan structure:
  - phases and milestones
  - dependency graph
  - required resources (tools, agents, permissions)
  - risk register and mitigations

## 2.2 Translation to governed work (plan → Orgo)
- Emit Orgo-ready work:
  - create/update **Case(s)**
  - create **Task graph** (atomic tasks with clear outputs)
  - assign routing labels, priority, owners, and policies

## 2.3 Execution design (how the work will be done)
- Select execution method for each task:
  - agent/tool/human
  - parameters and constraints
  - required runtime pack capabilities
- Ensure each task declares:
  - required artifacts in/out
  - failure criteria and retry policy
  - telemetry expectations (correlation IDs)

## 2.4 Change management (plan evolution)
- Plans can be revised, but revisions must be:
  - versioned
  - linked to their triggering event (new mandate, new canon, failed execution, feedback)
  - recorded as governed work updates (Orgo)

---

## 3) Interfaces and boundaries (contracts)

## 3.1 Primary upstream dependencies
- **Mandate**: purpose, scope, constraints, success criteria.
- **Blueprint**: schemas, policies, allowed tools/agents, routing taxonomy.
- **Kristal**: canonical truth (Exchange) and/or a Runtime Pack reference.
- **EkoH (optional)**: trust/impact signals used as prioritization inputs (never direct canon mutation).

## 3.2 Downstream consumers
- **Orgo**: receives Plan Envelope(s) and Task Envelope(s).
- **SwarmCraft**: receives tasks and executes them.
- **Konnaxion (optional)**: surfaces plan summaries for collaboration and feedback.

## 3.3 Hard boundary rules
- Architect-Strategy MUST NOT modify Kristal Exchange.
- Architect-Strategy MUST NOT claim new factual truth; it may only reference canon or mark content explicitly as a proposal/assumption.
- Architect-Strategy MUST NOT bypass Orgo gates; all work must enter execution through Orgo Tasks.
- Architect-Strategy MUST NOT generate user-facing final outputs (Architect-Render does that under stricter constraints).

---

## 4) Core artifacts produced

## 4.1 Plan Envelope (required)
A Plan Envelope is the canonical output of Architect-Strategy.

Minimum fields:
- `plan_id` (deterministic if configured; otherwise stable UUID + recorded seed/context)
- `mandate_id` and `mandate_version`
- `blueprint_version`
- `source_kristal_id` and/or `pack_id` (pinned)
- `objectives[]` (with acceptance criteria)
- `constraints[]` (explicit)
- `assumptions[]` (explicit; each has a status and optional evidence pointer)
- `milestones[]` (time/sequence, dependencies)
- `task_graph_ref` (links to Orgo tasks)
- `risk_register[]` (risk, severity, mitigations, owner)
- `justification_refs[]` (pointers into canon: IDs, sections, claim references)
- `created_at`, `created_by`, `strategy_engine_version`

## 4.2 Task Graph (Orgo-native)
Architect-Strategy emits a set of Tasks (and optionally a Case):
- Task is the atomic unit of work.
- Each task declares:
  - `task_id`
  - `inputs[]` (artifact IDs)
  - `expected_outputs[]` (artifact types/IDs)
  - `method` (agent/tool/human)
  - `constraints[]`
  - `retry_policy`
  - `telemetry_contract` (required logs/metrics)

## 4.3 Decision Notes (recommended)
- A compact “why this plan” appendix:
  - alternative options considered
  - rationale and trade-offs
  - explicit boundaries (“will not do”)

---

## 5) Determinism and auditability

Architect-Strategy may use heuristic or model-based reasoning. Because of that:

### Minimum requirement: auditability
Every plan MUST record enough context to explain and reproduce intent:
- pinned mandate, blueprint, canon/pack IDs
- engine/version identifiers and configuration hashes
- inputs used (artifact IDs)
- decision notes / justification refs

### Optional requirement: deterministic mode
If “deterministic mode” is enabled:
- plan structure and task graph MUST be reproducible given the same inputs + configuration
- any randomness must be seeded and recorded

---

## 6) Gates (validation rules)

Before emitting Orgo Tasks, Architect-Strategy MUST ensure:
1) Mandate exists and is active.
2) Blueprint version is pinned and compatible.
3) Canon/pack IDs are pinned (no floating “latest”).
4) Every task has:
   - a declared output artifact or measurable state change
   - explicit stop conditions and failure criteria
5) No task requests forbidden operations per mandate/policy.

If any gate fails:
- do not emit executable tasks
- emit a plan draft with blocking issues listed (or open an Orgo “blocked” Case).

---

## 7) Failure modes (expected behavior)

- Missing mandate/blueprint: refuse planning; return deterministic error code.
- Unpinned inputs (floating latest): refuse; require explicit pin.
- Conflicting constraints: produce a “conflict report” and request resolution via governed workflow.
- Tool capability mismatch (runtime pack lacks capability): produce a revised plan or mark tasks incompatible.
- Feedback contradicts assumptions: open a new Orgo Case/Task for reconciliation; do not mutate canon.

---

## 8) Observability (minimum)

Architect-Strategy MUST emit:
- `plan_id`, `mandate_id`, `build_id` (if applicable), `source_kristal_id` / `pack_id`
- `strategy_engine_version`, `config_hash`
- plan generation duration
- task counts by type (agent/tool/human)
- reasons for refusal/blocked plans (structured error codes)
- linkages to created Orgo Case/Task IDs

Recommended metrics:
- plan-to-execution success rate
- task failure clusters by capability/policy
- revision frequency and common triggers
- time-to-first-executable-task

---

## 9) Conformance tests (required)

A conformant Architect-Strategy implementation MUST provide tests for:
- refusing to plan without an active mandate
- refusing to plan with unpinned canon/pack inputs
- producing Orgo Tasks with required fields (inputs/outputs/method/constraints)
- never mutating canon (no write path to Kristal Exchange)
- producing explicit assumptions and constraints
- deterministic-mode reproducibility (if supported)
- producing a conflict report when constraints are unsatisfiable

---

## 10) Appendix: why Netzach is “Strategy”

This node embodies direction, initiative, and decomposition into action—yet it remains governed by:
- the mandate (Why/limits),
- the blueprint (How/allowed structure),
- and canon (What is true).

Architect-Strategy is the planner that makes action possible without becoming the authority on truth or execution.
