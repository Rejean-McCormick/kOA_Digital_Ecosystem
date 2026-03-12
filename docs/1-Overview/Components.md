# Components

This page is a map of the ecosystem’s components, their responsibilities, and how they fit together. It is intentionally **not** a node spec and does **not** restate Kristal artifact schemas.

## Planes (system at a glance)

- **Control plane:** Orgo (governance, orchestration, audit)
- **Truth plane (canonical):** Kristal (Exchange + Runtime Pack)
- **Resolution plane:** SenTient (Claim-IR → Resolved Claim-IR)
- **Distribution + interface plane:** Konnaxion (verify/activate/rollback; offline delivery + navigation)
- **Articulation plane:** Architect (Strategy + Render; deterministic outputs + trace)
- **Execution plane:** SwarmCraft (governed execution + telemetry)
- **Trust + impact plane (optional):** EkoH (signals/ledger; never mutates canon)

## Where components sit in the lifecycle

```mermaid
flowchart LR
  A[Chokmah<br/>Ingest + provenance] --> B[Binah<br/>Blueprint planning]
  B --> C[SenTient<br/>Resolve ambiguity]
  C --> D[Kristal<br/>Compile canonical truth]
  D --> E[Konnaxion<br/>Distribute + verify + activate]
  E --> F[Malkuth<br/>Runtime query/serve]
  F --> G[Architect<br/>Render with trace]
  G --> H[SwarmCraft<br/>Execute governed tasks]
  H --> I[Feedback -> Orgo<br/>New governed work]
````

## Component index (what to read next)

| Component               | What it does (high-level)                                                                                                    | Wiki page                                                            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **Orgo**                | Enforces stage order + hard gates; records operational evidence; drives releases/rollbacks                                   | [Components-Orgo](Components-Orgo)                                   |
| **Chokmah**             | Ingest boundary: turns raw inputs into immutable, provenance-pinned snapshots                                                | [Components-Chokmah](Components-Chokmah)                             |
| **Binah**               | Planning: converts mandate + available inputs into an auditable Blueprint (what will be built and how)                       | [Components-Binah](Components-Binah)                                 |
| **SenTient**            | Resolution: produces explicit, deterministic resolution outputs while preserving ambiguity when needed                       | [Components-SenTient](Components-SenTient)                           |
| **Kristal + Daat**      | Truth compilation + bridge: compile canonical Exchange + Runtime Pack; enforce pinned Kristal contracts at boundaries        | [Components-Kristal-and-Daat](Components-Kristal-and-Daat)           |
| **Konnaxion + Malkuth** | Distribution/runtime: verify-before-activate (fail-closed), atomic activation, deterministic rollback; serve offline queries | [Components-Konnaxion-and-Malkuth](Components-Konnaxion-and-Malkuth) |
| **Architect**           | Strategy + Render: propose governed work; render deterministic user-facing outputs with trace and “no new facts”             | [Components-Architect](Components-Architect)                         |
| **SwarmCraft**          | Execution: runs governed tasks, emits telemetry, does not mutate canonical truth directly                                    | [Components-SwarmCraft](Components-SwarmCraft)                       |

## Boundaries (what crosses between components)

kOA components exchange **typed artifacts**, not implicit state. The main pipeline carries (by type): input snapshots, claim proposals, resolution outputs, validation evidence, canonical truth artifacts, derived runtime packs, render outputs with trace, and kOA operational artifacts (cases/tasks/records).

## Where the detailed contracts live

* Node-by-node interface specs: see [Architecture / Nodes](Components-%28Nodes%29) (and each node page)
* kOA-owned operational artifacts: see [Artifacts](Artifacts)
* Kristal artifact contracts/schemas: see [Kristal v4 integration](Kristal-v4-integration) (pinned references; not duplicated here)


