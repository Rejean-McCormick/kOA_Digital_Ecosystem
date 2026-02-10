# ADR-0011: Component Splitting — Architect-Strategy vs Architect-Render

- **Status:** Accepted
- **Date:** 2026-02-09
- **Owners:** Architect Working Group
- **Supersedes:** None
- **Superseded by:** None

---

## Context

The ecosystem must support:
- planning and orchestration (work decomposition, routing, tool selection)
- user-facing articulation (reports, pages, answers)

These functions have incompatible correctness constraints:

- Planning can be exploratory and propose actions.
- Rendering must be deterministic and **must not introduce new facts**.
- If a single component performs both, the interface boundary becomes ambiguous and “truth drift” can occur (plans or speculative text leaking into user-facing deliverables).

The architecture establishes a truth boundary (canon pivot at Kristal). Rendering is downstream of that boundary and must therefore operate under a strict contract.

---

## Decision

Split Architect into **two separate contracts**:

### 1) Architect-Strategy (Netzach)
A planning/orchestration component that:
- produces **Plans** that are representable as Orgo work items (Cases/Tasks)
- selects tools and orchestrates multi-step workflows
- consumes mandate + policies + current canon
- MAY propose new work, hypotheses, questions, and follow-up extraction cycles

Architect-Strategy outputs are **not canonical truth** and MUST NOT be treated as truth artifacts.

### 2) Architect-Render (Hod)
A deterministic articulation component that:
- consumes **validated canonical artifacts** (Kristal Exchange and/or Runtime Pack, plus pipeline references)
- produces a **Render Bundle** containing a **trace_map**
- enforces “no new facts downstream”
- produces deterministic outputs for the same inputs + parameters
- omits, marks uncertain, or refuses deterministically when trace coverage is insufficient (policy-defined)

**Terminology alignment:** the canonical output artifact is **Render Bundle**. The canonical identifier field for that artifact MUST match the Render Bundle schema (e.g., `bundle_id` if that is the schema’s normative name). ADR-0006 and the Render Bundle schema MUST use the same identifier name.

Architect-Render outputs are the only user-facing “final outputs” permitted at system boundaries.

---

## Rationale

- Prevents mixing exploratory planning with truth articulation.
- Makes “no new facts” enforceable and testable via trace coverage rules.
- Simplifies integration: UIs consume Render Bundles; Orgo consumes Plans/Tasks.
- Enables different implementations: Strategy may be LLM-heavy; Render must be deterministic and traceable.

---

## Consequences

### Positive
- Clear boundary between planning and articulation.
- Stronger auditability and conformance testing.
- Reduced risk of hallucination drift in user-facing outputs.
- Enables caching and offline parity for render outputs.

### Costs / Trade-offs
- Additional interface and artifact definitions (Plan Envelope vs Render Bundle).
- More documentation and testing overhead.
- Requires explicit routing:
  - Strategy → Orgo → SwarmCraft
  - Render → Konnaxion/UI

---

## Alternatives considered

1) **Single Architect with internal modes**
- Rejected: boundary ambiguity, higher drift risk, harder to test.

2) **Render as a thin UI-only layer**
- Rejected: does not guarantee determinism or trace; cannot enforce “no new facts.”

3) **Treat Strategy outputs as “soft truth”**
- Rejected: violates truth boundary and encourages canon contamination.

---

## Follow-ups

- Define the canonical **Plan Envelope** schema produced by Architect-Strategy.
- Define policy profiles for Strategy behavior (when to propose a new extraction cycle).
- Ensure conformance tests cover:
  - Strategy never produces Render Bundles
  - Render never produces Plans/Tasks
  - Render enforces trace_map coverage and no-new-facts under adversarial templates
- Normalize terminology across ADR-0006, `docs/04-artifacts-contracts/07-render-bundle.md`, and `docs/04-artifacts-contracts/schemas/render-bundle.schema.json`:
  - identifier field (`render_id` vs `bundle_id`)
  - any required input reference field names

---

## Links

- `docs/03-nodes/07-netzach-architect-strategy.md`
- `docs/03-nodes/08-hod-architect-render.md`
- `docs/04-artifacts-contracts/07-render-bundle.md`
- `docs/07-implementation-guides/03-testing-conformance.md`
