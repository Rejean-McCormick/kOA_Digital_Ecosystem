# System at a Glance

**Purpose:** One-page mental model of the kOA ecosystem: what it is, what flows through it, and where “truth” lives.

## What this system does

kOA turns messy inputs into **validated, canonical knowledge** and then safely distributes and uses that knowledge in offline-capable products and workflows.

## The stage spine (end-to-end)

1. **Ingest** raw inputs (snapshots + provenance)
2. **Extract** structured proposals (claims)
3. **Resolve** ambiguity (entities, properties, literals)
4. **Validate** deterministically (accept/reject with a report)
5. **Compile** canonical knowledge + a portable offline pack
6. **Distribute** packs with fail-closed verification
7. **Render** deterministic user-facing output with trace coverage
8. **Execute** work (tasks) with telemetry
9. **Feedback** becomes new governed work (never mutates canon)

## Core components (who does what)

* **Orgo (control plane):** orchestrates stages, enforces gates, records operational evidence, drives releases.
* **SenTient (resolver):** turns ambiguous surfaces into explicit, typed resolution outputs (keeps ambiguity explicit when unresolved).
* **Kristal (truth pivot):** compiles canonical truth artifacts (Exchange) and derived offline artifacts (Runtime Pack).
* **Konnaxion (distribution + platform):** verifies/activates/rolls back packs; powers offline-first delivery and product navigation.
* **Architect (renderer):** produces deterministic natural-language (or other outputs) that cannot introduce new facts and must trace.
* **SwarmCraft (execution):** executes tasks into deliverables under constraints; emits telemetry.

## Artifact families (what crosses boundaries)

* **Pre-truth artifacts:** snapshots, claim proposals, resolution outputs, validation reports
* **Canonical truth artifacts:** the compiled knowledge source of truth
* **Derived distribution artifacts:** portable offline packs + indexes
* **Operational artifacts:** cases/tasks, build/release records, activation/rollback records, telemetry events

## Where “truth” lives

* “Truth” exists only as **typed artifacts produced after deterministic validation + compilation**.
* Downstream systems (rendering, execution, social/feedback) must not invent or mutate canonical truth.

## What this repo is responsible for

This repo documents:

* kOA’s **system behavior**, responsibilities, operations, and kOA-owned artifacts (workflow + distribution + execution).
* The **integration stance**: how kOA depends on Kristal (pinned version, compatibility expectations, conformance checks).

This repo does **not** re-specify Kristal artifact formats/schemas.

## Next reads

* `docs/10-system/architecture.md`
* `docs/10-system/lifecycle.md`
* `docs/20-nodes/index.md`
* `docs/40-integration/kristal-v4/index.md`
* `docs/50-operations/pipeline.md`
