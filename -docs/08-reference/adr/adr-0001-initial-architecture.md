# ADR-0001: Initial Architecture (Nodes, Paths, and Canon Pivot)

- **Status:** Accepted
- **Date:** 2026-02-09
- **Owners:** Ecosystem Architecture
- **Supersedes:** None
- **Superseded by:** None

---

## Context

We need a system that:
- converts heterogeneous inputs into **canonical truth artifacts**
- produces **portable offline runtime artifacts** for distribution and execution
- prevents drift, hallucination, and silent mutation of canon
- provides auditability and reproducibility across the full lifecycle
- scales across tenants, domains, and implementations without losing guarantees

A purely symbolic description is insufficient as a specification. The architecture must be defined by enforceable engineering constraints: contracts, schemas, gates, determinism, and integrity verification.

---

## Decision

Adopt a contract-driven architecture defined by:

### 1) Nodes + Paths model
Define the ecosystem as a set of **functional nodes** connected by **typed protocol paths**.

- **Nodes** are responsibility-bounded components with explicit inputs/outputs, invariants, failure modes, and observability.
- **Paths** specify what artifacts may cross boundaries and what behaviors are forbidden.

### 2) Canon pivot (Da’at / Kristal)
Introduce a single **canon pivot** where truth becomes canonical:

- Canonical truth exists only as **Kristal Exchange** artifacts produced by compilation.
- Downstream components MUST operate on compiled artifacts or refuse.

### 3) Typed artifact spine
Make the system operate by exchanging a small set of canonical artifacts:

- Input Snapshots
- Claim-IR
- Resolved Claim-IR
- Validation Report
- Kristal Exchange (+ Exchange Manifest)
- Runtime Pack (+ Pack Manifest)
- Render Bundle (+ trace_map)

Artifacts are versioned, schema-defined, and content-addressed where applicable.

### 4) Deterministic stage gates (Orgo control plane)
Use a control plane (Orgo) to enforce a deterministic stage sequence and hard gates:

- Stage order: Ingest → Extract → Resolve → Validate → Compile Exchange → Compile Pack → Publish/Distribute
- **Hard gate:** “No compile on fail” (validation failure blocks compilation)
- Build Records bind inputs/config/policies to outputs for audit and replay

### 5) Deterministic rendering with trace and “no new facts”
Split Architect into two contracts and enforce a hard rendering constraint:

- Architect-Strategy: planning/orchestration
- Architect-Render: deterministic articulation
- Render outputs MUST be delivered as **Render Bundles** with a **trace_map** and MUST NOT introduce new facts.

**Terminology alignment:** the canonical Render Bundle identifier field MUST match the Render Bundle schema (e.g., `bundle_id` if that is the schema’s normative name). ADR-0006, the Render Bundle schema, and the artifact contract doc MUST use the same identifier name.

### 6) Fail-closed distribution and rollback
Distribution MUST be safe by default:

- Verify manifests/payload hashes/signatures before activation.
- Fail closed on any verification mismatch.
- Support deterministic rollback to last-known-good.

### 7) Separate trust ledger (EkoH) that never mutates canon
Trust/impact scoring is captured in a ledger:

- append-only history, privacy-aware views
- weighting services (votes/influence) are derived signals
- ledger updates MUST be driven by validated outcomes
- ledger does not rewrite canonical truth

---

## Rationale

- **Contract-driven** interfaces prevent metaphor drift and make correctness testable.
- A single **canon pivot** eliminates ambiguity about what counts as truth.
- **Typed artifacts** allow interoperability across implementations and toolchains.
- **Deterministic gates** ensure reproducibility and prevent accidental canon mutation.
- **No new facts downstream** prevents rendering layers from becoming truth engines.
- **Fail-closed distribution** protects offline execution and user environments.
- A separate **ledger** captures social trust without corrupting canon.

---

## Consequences

### Positive
- End-to-end auditability: every output can be traced to validated inputs.
- Reproducible builds: same snapshots + config → same outputs (under determinism constraints).
- Clear separation of concerns: extraction, resolution, validation, compilation, rendering, execution, distribution.
- Safer offline operation: packs are verifiable and rollback-safe.
- Documentation becomes modular: each node/path/artifact can be specified independently.

### Costs / Trade-offs
- Higher upfront specification burden (schemas, contracts, error codes, conformance suite).
- More rigid integration requirements (components must conform or be isolated).
- Determinism constraints may limit certain probabilistic techniques unless upstream artifacts are frozen.

---

## Alternatives considered

1) **Mutable wiki-style canon**
- Rejected: weak auditability, high drift risk, unclear truth boundary.

2) **Agent-centric truth (outputs become truth)**
- Rejected: violates “no new facts,” difficult to reproduce, encourages hallucination drift.

3) **Single monolith without artifacts**
- Rejected: poor interoperability and weak boundary enforcement.

---

## Follow-ups (tracked in subsequent ADRs)

- ADR-0002: Truth boundary and canonicalization semantics
- ADR-0003: Deterministic gates and “no compile on fail”
- ADR-0004: Claim-IR and resolution model
- ADR-0005: Exchange and Runtime Pack formats
- ADR-0006: Render Bundle and trace_map requirements
- ADR-0007: Distribution verification and rollback
- ADR-0008: Orgo Cases/Tasks and routing taxonomy
- ADR-0009: SwarmCraft telemetry contracts
- ADR-0010: EkoH ledger and voting model

---

## Links

- `docs/01-principles/01-invariants.md`
- `docs/02-architecture/01-system-overview.md`
- `docs/02-architecture/02-lifecycle.md`
- `docs/06-operations/01-build-pipeline.md`
- `docs/07-implementation-guides/03-testing-conformance.md`
