# System Overview

This document defines the ecosystem as a **contract-driven knowledge system**: it converts raw inputs into **canonical truth artifacts** (Kristal Exchange) and **portable offline Runtime Packs**, then distributes, renders, and operationalizes them through deterministic governance, strict interfaces, and end-to-end traceability.

The system is described as a set of **functional nodes** (components with responsibilities and invariants) connected by **protocol paths** (typed boundaries). Da’at (Kristal) is the **truth pivot**: the point at which information becomes canonical.

---

## 1) System purpose

### What the system does
- Ingests heterogeneous sources (text, media, feeds, human inputs).
- Extracts structured claims (Claim-IR) from raw inputs.
- Resolves claims into coherent, evidence-linked statements (Resolved Claim-IR) while preserving ambiguity where needed.
- Validates deterministically and compiles validated truth into:
  - **Kristal Exchange** (canonical truth store)
  - **Runtime Pack** (portable, offline-executable derivative)
- Distributes packs safely (verification before activation, deterministic rollback).
- Renders user-facing outputs deterministically with **trace maps** and **no new facts**.
- Executes tasks via SwarmCraft (agents/tools/humans) with telemetry.
- Captures trust/impact in a ledger (EkoH) without mutating canon.

### What the system is NOT
- Not a “chat memory” system where outputs become truth by repetition.
- Not a freeform generative pipeline where rendering can invent missing information.
- Not a mutable wiki where edits directly rewrite canonical truth without governance.

---

## 2) Core architecture concept: nodes + paths + canon pivot

### Nodes
A **node** is a functional module with:
- Responsibilities (what it must do)
- Inputs/outputs (typed artifacts)
- Invariants (rules it must never violate)
- Failure modes + observability

### Paths
A **path** is a protocol boundary:
- Defines which artifacts can cross
- Defines forbidden behaviors (e.g., “no new facts” downstream)
- Defines verification requirements and error handling

### Canon pivot (Da’at)
Da’at is the “truth gate”:
- Nothing is canonical until it passes validation and compilation into Kristal Exchange.
- Downstream components operate on compiled artifacts, not on raw guesses.

---

## 3) Global invariants (summary)

These rules apply across the entire ecosystem:

1. **Truth boundary**: only validated + compiled artifacts become canonical.
2. **Typed interfaces**: all cross-node exchange uses explicit, versioned artifacts.
3. **Deterministic gates**: stage ordering and “no compile on fail.”
4. **No new facts downstream**: rendering/articulation cannot invent; it must trace.
5. **Fail-closed distribution**: verify before activation; rollback deterministically.
6. **Governed feedback**: feedback creates new work; it does not mutate canon directly.
7. **End-to-end traceability**: every assertion must be traceable to validated sources.

The detailed list and rationale live in `docs/01-principles/`.

---

## 4) Functional node map (Sephiroth-style, technical)

This is the canonical node set for the ecosystem documentation. Each node has a full spec in `docs/03-nodes/`.

### Node summary table

| Node | Functional name | Primary responsibility | Primary outputs |
|------|------------------|------------------------|-----------------|
| Keter | Mandate | Defines purpose, scope, constraints, success | Mandate bundle |
| Chokmah | Inputs | Captures raw sources with provenance | Input snapshots |
| Binah | Blueprint | Defines schemas, ontologies, policies | Schema/policy bundles |
| Chesed | Connectivity (Konnaxion) | Social layer + distribution surfaces | Access/roles, delivery state |
| Gevurah | Governance (Orgo) | Control plane, gating, workflow | Cases/Tasks, build records |
| Tiferet | Resolution (SenTient) | Resolves claims; preserves ambiguity | Resolved Claim-IR |
| Netzach | Strategy (Architect-Strategy) | Planning, decomposition, orchestration | Plans, tool/task graphs |
| Hod | Articulation (Architect-Render) | Deterministic rendering + trace map | Render bundles |
| Yesod | Execution (SwarmCraft) | Executes tasks via agents/tools/humans | Results + telemetry |
| Malkuth | Ledger (EkoH) | Trust/impact scoring + audit trail | Scores, votes, histories |
| Da’at | Kristal | Canonicalization + portability | Exchange + Runtime Pack |

### Split contracts (important)
- **Architect** is two contracts:
  - Architect-Strategy (planning/orchestration)
  - Architect-Render (deterministic articulation with trace, no new facts)
- **Konnaxion** often splits into:
  - Social facet (collaboration surface)
  - Distribution facet (pack verification/activation/rollback)

---

## 5) Canonical artifact spine (what moves through the system)

The ecosystem is held together by a small set of canonical artifacts:

1. **Input Snapshot**
   - Immutable capture of raw sources + provenance
2. **Claim-IR**
   - Structured extracted claims (schema-constrained; includes evidence pointers)
3. **Resolved Claim-IR**
   - Harmonized claims (may preserve ambiguity explicitly)
4. **Validation Report**
   - Deterministic acceptance/rejection; reasons and rule references
5. **Kristal Exchange**
   - Canonical truth store (content-addressed, immutable, signed)
6. **Runtime Pack**
   - Portable offline pack derived from Exchange (signed, verifiable)
7. **Render Bundle**
   - Rendered output + metadata + `trace_map` coverage

Artifact definitions live in `docs/04-artifacts-contracts/`.

---

## 6) Lifecycle at a glance (stage spine)

This is the reference end-to-end flow (detailed in `docs/02-architecture/02-lifecycle.md`):

1. **Ingest** inputs → snapshot provenance (Inputs)
2. **Extract** → Claim-IR (extractors)
3. **Resolve** → Resolved Claim-IR (SenTient)
4. **Validate** deterministically (Orgo gate)
5. **Compile** → Kristal Exchange + Runtime Pack (Kristal)
6. **Distribute** pack (Konnaxion: verify → activate → rollback-safe)
7. **Render** outputs (Architect-Render: deterministic + trace_map, no new facts)
8. **Execute** tasks (SwarmCraft: telemetry + results)
9. **Feedback** → new governed work (Orgo Cases/Tasks)

---

## 7) Control plane: Orgo

Orgo is the governance backbone:
- Defines workflow stages and gates
- Manages Cases/Tasks as canonical units of work
- Enforces “no compile on fail”
- Produces audit-grade build records binding inputs → configs → outputs

Orgo details live in:
- `docs/03-nodes/05-gevurah-orgo.md`
- `docs/06-operations/01-build-pipeline.md`

---

## 8) Distribution: Konnaxion (runtime delivery)

Konnaxion ensures safe offline delivery:
- Verifies packs before activation (fail-closed)
- Manages caching, compatibility checks, and deterministic rollback
- Tracks installed/active versions and last-known-good state

Konnaxion details live in:
- `docs/03-nodes/04-chesed-konnaxion.md`
- `docs/06-operations/02-release-management.md`

---

## 9) Rendering: Architect-Render (deterministic articulation)

Architect-Render is a hard boundary:
- Cannot introduce new facts
- Must output deterministic results from the same validated inputs + parameters
- Must produce a `trace_map` covering factual assertions
- Must refuse deterministically if trace coverage is insufficient

Rendering details live in:
- `docs/03-nodes/08-hod-architect-render.md`
- `docs/04-artifacts-contracts/07-render-bundle.md`

---

## 10) Execution: SwarmCraft

SwarmCraft executes tasks using tools, agents, and/or humans:
- Treat as an archetype (multiple implementations possible)
- Must emit telemetry (correlation IDs, status, artifacts produced)
- Must not mutate canon directly; outputs become inputs to governed cycles

Execution details live in:
- `docs/03-nodes/09-yesod-swarmcraft.md`
- `docs/06-operations/03-observability.md`

---

## 11) Ledger: EkoH (trust and impact)

EkoH captures trust signals without rewriting truth:
- Expertise/ethics scoring and history
- Voting/aggregation primitives
- Auditability and privacy boundaries

Ledger details live in:
- `docs/03-nodes/10-malkuth-ekoh-ledger.md`
- `docs/04-artifacts-contracts/09-ekoh-scores-votes.md`

---

## 12) Minimal note on resonance (non-authoritative)

The ecosystem can be explained using symbolic lenses (node semantics and cyclic rigor), but correctness is defined by contracts: invariants, typed artifacts, and deterministic gates. This note is informational only; the normative spec is the contract set.

(See `docs/02-architecture/04-parallel-note.md`.)

---

## 13) Where to go next

- Read principles:
  - `docs/01-principles/01-invariants.md`
  - `docs/01-principles/02-truth-boundary.md`
  - `docs/01-principles/03-determinism.md`
- Follow the lifecycle:
  - `docs/02-architecture/02-lifecycle.md`
- Dive into node specs:
  - `docs/03-nodes/`
- Implement contracts:
  - `docs/04-artifacts-contracts/`
  - `docs/05-protocols-flows/`
