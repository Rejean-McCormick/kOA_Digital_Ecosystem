# Documentation Scope

**Normative for kOA:** YES  
**External normative reference:** Kristal v4 (pinned)

This documentation describes the **kOA Digital Ecosystem**: a contract-driven pipeline and operating model that turns inputs into **validated knowledge artifacts** (via Kristal) and then distributes, renders, executes, and governs outcomes across the ecosystem.

## Goals

- Provide a clear, implementable understanding of **kOA components**, their responsibilities, and their interfaces.
- Define **kOA-owned contracts** (Orgo workflow objects, Build/Release records, Konnaxion operational state, etc.).
- Specify **operational behavior** (gates, safety, rollback, observability, incident response).
- Make integration with **Kristal v4** explicit **without duplicating** Kristal’s normative contracts.

## Non-goals

- Re-stating or re-implementing Kristal’s normative artifact definitions (schemas, canonicalization, hashing, signing targets).
- Providing product marketing, UX copy, or user manuals for specific applications.
- Defining deployment topology, cloud-specific infrastructure, or vendor-specific operational procedures beyond general guidance.

## Normative boundaries (ownership)

### Kristal-owned (external normative source)
Kristal defines the normative contracts for:
- Claim-IR, Resolved Claim-IR, Validation Report
- Exchange Manifest / Exchange identity rules
- Runtime Pack Manifest and offline execution constraints
- Canonicalization identifiers and integrity envelope shapes (hash/signature rules)

kOA docs must **reference** these, not duplicate them. Use:
- `40-integration/kristal-v4/pinned-dependency.md` (pin + required paths)
- `40-integration/kristal-v4/contract-pointers.md` (links to the exact pinned schemas/spec)

### kOA-owned (normative here)
kOA defines the normative contracts for:
- Orgo governance objects (Cases/Tasks), routing taxonomy, and workflow stage gating
- Build/Release records (operational evidence linking inputs → outputs)
- Konnaxion distribution/activation/rollback state and operational signals
- Ecosystem-level invariants (truth boundary enforcement, fail-closed policies, deterministic gate behavior)

## What “no redundancy” means in practice

- If a page needs to mention a Kristal artifact, it should:
  1) name the artifact and where it sits in the pipeline,
  2) specify **kOA-specific requirements around it** (e.g., “Orgo must block publish on failed verification”),
  3) link to the pinned Kristal spec/schema for field-level definitions.

- kOA docs should never carry “illustrative manifests” for Kristal artifacts unless they are **verbatim** copied from the pinned Kristal version and clearly marked as such (preferred: link instead).

## Audiences

- **Implementers**: building services that produce/consume artifacts and participate in gates.
- **Integrators**: connecting external systems to ingestion, distribution, or rendering.
- **Operators**: running builds, releases, rollbacks, incident response, and audits.
- **Architects**: evolving contracts and invariants through ADRs.

## Change control

Any change to:
- an invariant,
- a kOA-owned artifact schema,
- a gate rule or safety policy,
must be accompanied by an ADR and a conformance test update (where applicable).

Changes to Kristal contracts must happen in the pinned Kristal source; then kOA updates its integration profile/conformance docs accordingly.

## Suggested reading order

1. `index.md`
2. `00-overview/system-at-a-glance.md`
3. `10-system/` (architecture/lifecycle/components/boundaries)
4. `20-nodes/` (node-by-node interface specs)
5. `30-artifacts/` (kOA-owned artifacts + schemas)
6. `40-integration/kristal-v4/` (pinned dependency + pointers + kOA profile + conformance)
7. `50-operations/` and `60-guides/` (how to run and build)