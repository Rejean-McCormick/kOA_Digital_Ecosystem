# ADR-0004: Architect split (Strategy vs Render)

**Status:** Accepted  
**Date:** 2026-02-27  
**Decision Owner:** kOA Ecosystem Architecture  
**Scope:** Component boundaries, determinism guarantees, and responsibility separation for the Architect role.

---

## 1) Context

The ecosystem requires:
- a strict truth boundary (canonical truth only after validation + compilation),
- deterministic downstream behavior (rendering and execution must not introduce new facts),
- clear governance over work creation and lifecycle (cases/tasks).

A single “Architect” component combining planning and rendering creates failure modes:
- plans may become implicitly treated as truth,
- rendering may vary based on planning heuristics,
- feedback loops can accidentally mutate canon or hide provenance.

We need explicit responsibility separation.

---

## 2) Decision

Split the Architect role into two components:

1) **Architect-Strategy (Netzach)**
- proposes work (plans → cases/tasks),
- performs prioritization, scheduling, decomposition,
- can use heuristic / probabilistic methods **upstream of truth**,
- does not publish canonical truth artifacts.

2) **Architect-Render (Hod)**
- produces user-facing outputs (Render Bundles),
- must be deterministic given pinned inputs + templates + parameters,
- must not introduce new facts; all factual statements must be traced or deterministically omitted/refused.

---

## 3) Boundaries and interfaces

### 3.1 Strategy → Orgo
- Strategy outputs **work proposals** (plan envelope / case/task creation requests).
- Orgo owns lifecycle state and auditing.
- Strategy must not mutate canon directly; it can only request work.

### 3.2 Render → Consumers
- Render consumes validated artifacts (via Kristal Exchange and/or Runtime Pack) and templates/params.
- Render emits a Render Bundle with a Trace Map sufficient for audit/verification.

---

## 4) Determinism and governance implications

- Strategy is allowed to be non-deterministic, but its outputs must be captured and governed as work objects (cases/tasks).
- Render must be deterministic and policy-driven; if policy disallows an output, Render must deterministically refuse/omit.

---

## 5) Consequences

### Positive
- Clear separation of “decide what to do” vs “explain what is true”.
- Stronger determinism guarantees for downstream artifacts.
- Reduced risk of silent canon mutation through planning behavior.

### Trade-offs
- Requires an explicit plan envelope / routing mechanism to Orgo.
- Requires clear template/parameter pinning for deterministic rendering.

---

## 6) Follow-ups

- Define the minimal Plan → Case/Task request format (kOA-owned artifact) if not already standardized.
- Ensure conformance tests include deterministic Render Bundle checks.
- Update node docs (`docs/20-nodes/netzach-architect-strategy.md`, `docs/20-nodes/hod-architect-render.md`) to reflect this split.

---