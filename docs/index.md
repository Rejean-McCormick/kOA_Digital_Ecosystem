# Digital Ecosystem — Documentation

This documentation describes a contract-driven ecosystem that turns raw inputs into **canonical knowledge artifacts** (Kristal Exchange) and **portable offline Runtime Packs**, then distributes, renders, and operationalizes them through strict governance, deterministic gates, and traceable outputs.

## What this ecosystem is

- **A governed pipeline**: raw inputs → structured claims → resolution → validation → canonicalization → distribution → rendering → execution → feedback.
- **A truth system**: “truth” exists only as **typed artifacts** produced by deterministic gates.
- **A distribution system**: Runtime Packs are verified, activated, and rolled back safely (fail-closed).
- **A rendering system**: user-facing outputs are deterministic, traceable, and cannot introduce new facts.
- **An execution system**: tasks are executed by SwarmCraft instances with telemetry, producing governed feedback.

## Core principles (non-negotiable)

1. **Truth boundary**: only validated + compiled artifacts become canonical.
2. **Typed interfaces**: every boundary exchanges explicit artifacts (schemas/contracts).
3. **Deterministic gates**: stage ordering, “no compile on fail,” deterministic rendering.
4. **No new facts downstream**: rendering/articulation cannot invent; it must trace.
5. **Fail-closed distribution**: verification before activation; deterministic rollback.
6. **Governed feedback**: feedback creates new work; it does not mutate canon.

## Reading path

1. Start here:
   - [Reading Path](00-reading-path.md)
   - [Global Invariants](01-principles/01-invariants.md)
   - [Truth Boundary](01-principles/02-truth-boundary.md)
   - [Determinism](01-principles/03-determinism.md)

2. Understand the whole:
   - [System Overview](02-architecture/01-system-overview.md)
   - [Lifecycle](02-architecture/02-lifecycle.md)
   - [Components Map](02-architecture/03-components-map.md)

3. Dive into node specs:
   - [Nodes](03-nodes/)

4. Implement / integrate:
   - [Artifacts & Contracts](04-artifacts-contracts/)
   - [Protocols & Flows](05-protocols-flows/)
   - [Operations](06-operations/)
   - [Implementation Guides](07-implementation-guides/)

## Architecture at a glance (functional nodes)

Each node has a dedicated specification under `03-nodes/`:

- Mandate (Keter): `03-nodes/01-keter-mandate.md`
- Inputs (Chokmah): `03-nodes/02-chokmah-inputs.md`
- Blueprint (Binah): `03-nodes/03-binah-blueprint.md`
- Connectivity (Chesed / Konnaxion): `03-nodes/04-chesed-konnaxion.md`
- Governance (Gevurah / Orgo): `03-nodes/05-gevurah-orgo.md`
- Resolution (Tiferet / SenTient): `03-nodes/06-tiferet-sentient.md`
- Strategy (Netzach / Architect-Strategy): `03-nodes/07-netzach-architect-strategy.md`
- Articulation (Hod / Architect-Render): `03-nodes/08-hod-architect-render.md`
- Execution (Yesod / SwarmCraft): `03-nodes/09-yesod-swarmcraft.md`
- Ledger (Malkuth / EkoH): `03-nodes/10-malkuth-ekoh-ledger.md`
- Truth pivot (Da’at / Kristal): `03-nodes/11-daat-kristal.md`

## Canonical lifecycle (stage spine)

The reference stage spine is defined in:
- [Lifecycle](02-architecture/02-lifecycle.md)
- [Build Pipeline (Orgo)](06-operations/01-build-pipeline.md)

High-level flow:

1. Ingest inputs → snapshot provenance
2. Extract → Claim-IR
3. Resolve → Resolved Claim-IR
4. Validate (deterministic) → validation report
5. Compile → Kristal Exchange + Runtime Pack
6. Distribute → verify/activate/rollback
7. Render → deterministic output + trace map
8. Execute → tasks + telemetry
9. Feedback → new governed work

## Reference

- [Glossary](08-reference/glossary.md)
- [FAQ](08-reference/faq.md)
- [ADRs](08-reference/adr/) (architecture decision records)
- [Diagrams](08-reference/diagrams/)
