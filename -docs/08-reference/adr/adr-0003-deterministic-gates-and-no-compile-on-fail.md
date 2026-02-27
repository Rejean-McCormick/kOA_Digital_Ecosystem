# ADR-0003: Deterministic Gates and “No Compile on Fail”

## Status
Accepted (normative)

## Context
The ecosystem’s truth boundary is defined by a strict pipeline: ingest → extract → resolve → validate → compile → publish/distribute. Orgo is the control plane that enforces ordering and gating. :contentReference[oaicite:0]{index=0}

A core safety requirement is that canonical truth (Kristal Exchange) must never be produced from invalid or unvalidated material. The architecture achieves “equilibrium” via explicit gates, immutable artifacts, and non-mutating feedback loops. :contentReference[oaicite:1]{index=1}

## Decision
We adopt **deterministic acceptance gates** as first-class architecture primitives, with the following mandatory rule:

### D1 — “No compile on fail” is a hard gate
If validation fails, Orgo:
- MUST mark the workflow as failed at the Validation stage,
- MUST NOT invoke Exchange or Runtime Pack compilation,
- MUST surface the validation report (structured codes + messages) to operators. :contentReference[oaicite:2]{index=2}

### D2 — Stage ordering is mandatory
A Kristal build workflow MUST follow this stage order:
1. Ingest
2. Extract → Claim-IR
3. Resolve (SenTient) → Resolved Claim-IR
4. Validate
5. Compile → Exchange
6. Compile → Runtime Pack
7. Publish/Distribute
8. Post-publish verification (recommended) :contentReference[oaicite:3]{index=3}

Orgo MAY split stages into sub-stages, but MUST preserve ordering constraints and gating semantics. :contentReference[oaicite:4]{index=4}

### D3 — Deterministic stage outputs are recorded
For each stage, Orgo MUST record content-addressed references to outputs, including Claim-IR batch IDs, Resolved Claim-IR batch IDs, validation report IDs, Exchange `kristal_id`, Runtime Pack `pack_id` and payload hashes. :contentReference[oaicite:5]{index=5}

### D4 — Canonisation boundary inherits integrity constraints
At the canonisation boundary (validation → Kristal compilation), constraints include: “no compile on fail”; deterministic canonicalization; content-addressed identity; integrity checks fail-closed. :contentReference[oaicite:6]{index=6}

## Rationale
- Truth becomes canonical only after validation + compilation; therefore validation must be the deterministic acceptance gate. :contentReference[oaicite:7]{index=7}
- Validation failure must produce no Exchange update (“no compile on fail”)—this is explicitly a minimal conformance requirement. :contentReference[oaicite:8]{index=8}
- The system’s equilibrium depends on enforceable gates and immutable artifacts, not informal best-effort behavior. :contentReference[oaicite:9]{index=9}

## Consequences

### Positive
- Prevents invalid material from entering canonical truth (Exchange/Runtime Pack). :contentReference[oaicite:10]{index=10}
- Makes builds auditable and reproducible by requiring recorded content-addressed stage outputs. :contentReference[oaicite:11]{index=11}
- Establishes a clear operational truth boundary aligned with global invariants (immutability, truth boundary, fail-closed verification, auditability). :contentReference[oaicite:12]{index=12}

### Costs / Tradeoffs
- More workflows will stop early; operators must resolve validation errors instead of “shipping anyway.” :contentReference[oaicite:13]{index=13}
- Validators must be deterministic and produce stable, structured reports, or the gate becomes noisy/opaque. :contentReference[oaicite:14]{index=14}
- Requires strong build record discipline (IDs, hashes, provenance) to make deterministic replay meaningful. :contentReference[oaicite:15]{index=15}

## Alternatives considered (rejected)

### A1 — Best-effort compile with warnings
Rejected because it breaks the truth boundary and allows invalid data to become canonical, undermining immutability and auditability. :contentReference[oaicite:16]{index=16}

### A2 — Partial compilation (compile whatever passed)
Rejected because canonical artifacts must be whole, comparable, and reproducible; partial compilation creates ambiguous canonical state and complicates distribution integrity expectations. :contentReference[oaicite:17]{index=17}

### A3 — Manual override to bypass validation gate
Rejected as a default path. If any override is introduced later, it must be a governed workflow with explicit approvals and must still preserve auditability and artifact immutability (new build, new IDs). :contentReference[oaicite:18]{index=18}

## Implementation notes (normative)

### I1 — Workflow state
- Orgo MUST persist workflow stage state and stop the workflow on validation failure. :contentReference[oaicite:19]{index=19} :contentReference[oaicite:20]{index=20}

### I2 — Build records
For every build, Orgo MUST persist a Build Record including unique identifiers, input snapshots, compiler identity/version, config hash, and policy selections. :contentReference[oaicite:21]{index=21}

### I3 — Replay
Orgo SHOULD support deterministic replay: same input snapshots + build configuration should yield the same outputs (subject to probabilistic extractors being frozen or replaced with stored Claim-IR). Replay MUST NOT bypass integrity checks. :contentReference[oaicite:22]{index=22}

## References
- Orgo Workflow Contract (Kristal v3 Integration): stages + gating + build record requirements. :contentReference[oaicite:23]{index=23} :contentReference[oaicite:24]{index=24}
- Path D — Loi (Canonisation Boundary): constraints include “no compile on fail”. :contentReference[oaicite:25]{index=25}
- Global invariants and conformance suite examples: validation failure must produce no Exchange update. :contentReference[oaicite:26]{index=26}
- “Equilibre séphirotique” framing as implementable gates + immutable artifacts + non-mutating feedback. :contentReference[oaicite:27]{index=27}
