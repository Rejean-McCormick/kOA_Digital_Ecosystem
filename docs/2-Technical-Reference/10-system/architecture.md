# Architecture

**Normative for kOA:** YES (system architecture)  
**External normative references:** Kristal v4 (pinned) for Kristal artifact formats/schemas; kOA does not restate Kristal contracts

---

## 1) Purpose

Define the kOA ecosystem architecture as a **contract-driven pipeline** that turns raw inputs into **canonical knowledge artifacts** (via Kristal) and then distributes, renders, operationalizes, and governs them safely.

This document describes:
- the major components (nodes) and their boundaries,
- the control-plane vs data-plane split,
- the stage spine (lifecycle),
- cross-cutting properties (determinism, fail-closed verification, auditability, offline behavior).

It does **not** redefine Kristal schemas or artifact formats; those are external normative sources pinned under `docs/40-integration/kristal-v4/`.

---

## 2) Scope and non-goals

### In scope
- kOA nodes and responsibilities
- Stage ordering and gates (architecture-level, not schema-level)
- Operational flow and failure containment
- Where Kristal artifacts enter/exit kOA boundaries

### Out of scope (by design)
- Field-level schemas for Kristal artifacts (Claim-IR, Exchange, Runtime Pack, Validation Report, etc.)
- Canonicalization/signature/hashing mechanics beyond “what is required at boundaries”
- Full deployment topology details (documented in operations)

---

## 3) System at a glance

kOA is a governed ecosystem organized around one rule: **contracts and invariants define truth**.

High-level flow:
1. Ingest inputs → snapshot provenance
2. Extract → Claim-IR
3. Resolve → Resolved Claim-IR
4. Validate (deterministic) → Validation Report
5. Compile → Kristal Exchange + Runtime Pack
6. Distribute → verify/activate/rollback
7. Render → deterministic output + trace map
8. Execute → tasks + telemetry
9. Feedback → new governed work

---

## 4) Primary nodes (components)

### 4.1 Control plane (governance + orchestration)
- **Orgo**: workflow controller; enforces stage ordering, gating, audit records, and publication rules.

### 4.2 Knowledge compilation (truth substrate)
- **Kristal**: compiles validated knowledge into canonical Exchange and derived Runtime Packs. Kristal is the **truth pivot** inside the ecosystem.
- **SenTient**: reconciliation/resolution engine producing Resolved Claim-IR from Claim-IR, preserving ambiguity explicitly.

### 4.3 Distribution + runtime
- **Konnaxion (distribution facet)**: verifies, activates, and rolls back Runtime Packs **fail-closed**, with atomic activation and deterministic rollback.
- **Malkuth (runtime)**: offline serving and execution substrate consuming the active Runtime Pack (read-only), providing deterministic query and (optional) routine execution.

### 4.4 Rendering (human-facing outputs)
- **Architect-Render**: deterministic generation **after** validation; must not introduce new facts and must provide traceability coverage (Render Bundle + trace map) per the kOA contract.

### 4.5 Optional execution (work output)
- **SwarmCraft**: executes governed tasks and emits telemetry; does not mutate canonical truth directly.

---

## 5) Boundary contracts (what crosses interfaces)

kOA boundaries exchange **typed artifacts**, not implicit state.

At a minimum, the pipeline carries:
- Input snapshots (provenance-pinned)
- Claim-IR (proposal boundary; Kristal-defined)
- Resolved Claim-IR (resolution boundary; Kristal-defined)
- Validation Report (deterministic acceptance evidence; Kristal-defined)
- Kristal Exchange (canonical truth; Kristal-defined)
- Runtime Pack (offline query payload; Kristal-defined)
- Render Bundle (deterministic user-facing output + trace map; kOA-defined)
- Orgo Cases/Tasks and operational records (kOA-native)

**Normative note:** Kristal artifact schemas are owned externally; kOA references the pinned Kristal version rather than duplicating schema definitions.

---

## 6) Stage spine and gate semantics (architecture-level)

### 6.1 Mandatory stage ordering
Orgo enforces this order:
**Ingest → Extract → Resolve → Validate → Compile → Publish/Distribute**.

(“Compile” invokes Kristal to produce Exchange and Runtime Pack from validated inputs.)

### 6.2 Hard gates (fail-closed)
- If validation fails, Orgo must not compile Exchange or Runtime Pack (“no compile on fail”).
- If distribution verification fails (hashes/signatures/compat), activation must not proceed.

### 6.3 Immutability and governed change
- Canonical truth artifacts are immutable once published; changes produce new versions (no in-place mutation).
- Feedback creates new governed work (Cases/Tasks) and can trigger new builds, not direct mutation.

---

## 7) Cross-cutting properties

### 7.1 Determinism
Determinism is enforced by pinned inputs, pinned policies/config, and deterministic gates. kOA documents determinism as a system invariant; Kristal defines canonicalization and identity mechanics externally (pinned reference).

### 7.2 Auditability
Every stage output must be traceable via Orgo records and artifact references, enabling reproducibility and investigation.

### 7.3 Offline-first distribution
Runtime Packs are designed for offline execution; Konnaxion operationalizes offline verification/activation/rollback, and Malkuth serves the active pack offline.

### 7.4 Multi-tenancy and trust boundaries
Tenant-scoped policies and trust roots must be enforced at verification and publication boundaries (documented in operations/security; Kristal supplies authority/trust schema structures externally).

---

## 8) Where to go next

- Lifecycle: `docs/10-system/lifecycle.md`
- Components map: `docs/10-system/components.md`
- Trust boundaries: `docs/10-system/trust-boundaries.md`
- Determinism: `docs/10-system/determinism.md`
- Node specs: `docs/20-nodes/`
- Kristal integration (pinned references + kOA profile): `docs/40-integration/kristal-v4/`