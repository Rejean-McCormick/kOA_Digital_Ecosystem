# FAQ

## What is the ecosystem, in one sentence?
A governed pipeline that turns inputs into **validated canonical truth (Kristal Exchange)** and **portable execution artifacts (Runtime Packs)**, then distributes, renders, executes, and iterates through feedback **without mutating canon directly**.

---

## What is “canon” here?
Canon is **Kristal Exchange**: the authoritative, immutable truth artifact produced only after validation passes and compilation succeeds.

---

## Can anything edit canon directly?
No. Canon must never be edited in place. Truth-impacting changes must re-enter the pipeline as governed work and produce a new Exchange artifact.

---

## Where is the “truth boundary”?
At the transition: **Validated (Orgo) → Compiled (Kristal)**. Before that, everything is proposal/IR; after that, it is canon.

---

## What are the key artifacts?
- Input snapshots (raw, immutable)
- Claim-IR (extracted proposals)
- Resolved Claim-IR (post-resolution)
- Validation Report (gate decision)
- Kristal Exchange (canon)
- Runtime Pack (derived offline projection)
- Render Bundle (user-facing output + trace_map)
- Orgo Case/Task (governed work units)

See: `../04-artifacts-contracts/01-artifact-catalog.md`

---

## What does “contract-driven” mean in this system?
Correctness is defined by:
- typed artifacts (schemas),
- normative rules (MUST/MUST NOT),
- deterministic gates (validation/compile/verify/render rules),
- and conformance tests.

---

## What does “no new facts” mean?
Any user-facing output must not introduce factual statements that are not supported by upstream validated artifacts. If support is missing: omit, mark uncertainty (only if upstream encodes it), or refuse (strict mode).

See: `../04-artifacts-contracts/07-render-bundle.md`

---

## How is ambiguity handled?
Ambiguity is first-class:
- unresolved disputes remain explicit,
- ambiguity can be carried through IR and canon as structured uncertainty/alternatives,
- renderers must reflect it rather than collapsing it.

See: `../04-artifacts-contracts/03-resolved-claim-ir.md`

---

## What is Orgo?
Orgo is the **control plane**:
- governs Cases/Tasks,
- enforces gates and lifecycle,
- records build/release lineage and outcomes,
- routes work based on taxonomy and policy.

---

## What is Kristal?
Kristal is the **compiler**:
- produces Kristal Exchange (canon) + Runtime Pack (derived),
- binds inputs/versions/config in manifests,
- refuses to compile unless validation passes (“no compile on fail”).

See: `../03-nodes/11-daat-kristal.md`

---

## What is Konnaxion?
Konnaxion is the **edge**:
- social surfaces (identity/roles/collaboration),
- distribution surfaces (pack verification, activation, rollback, offline caching),
- collects feedback and operational signals without mutating canon.

See: `../03-nodes/04-chesed-konnaxion.md`

---

## What is SwarmCraft?
SwarmCraft is the **execution substrate**:
- executes Orgo Tasks using agents/tools/humans,
- emits telemetry and produces artifacts,
- never writes to canon.

---

## What is the difference between Architect-Strategy and Architect-Render?
- **Architect-Strategy**: planning and decomposition into governed work (plans → tasks).
- **Architect-Render**: deterministic articulation (outputs + trace_map) under “no new facts.”

See: `../03-nodes/07-netzach-architect-strategy.md` and `../04-artifacts-contracts/07-render-bundle.md`

---

## What is “fail-closed” and where does it apply?
Fail-closed means: if verification or prerequisites fail, the system rejects the operation rather than attempting a “best effort” that could corrupt correctness.

Applied at minimum to:
- validation → compilation (no compile on fail),
- pack verification → activation (no activate on verify failure),
- strict rendering (refuse on trace gap).

---

## How does feedback change the system?
Feedback becomes governed work:
- captured and normalized,
- triaged into Cases/Tasks,
- truth-impacting items re-enter the full pipeline to produce a new Exchange/Pack.

See: `../05-protocols-flows/04-feedback-path.md`

---

## What does “deterministic” mean here?
Given the same pinned inputs, versions, and configuration:
- validation decisions must match,
- compilation outputs and IDs must match (in deterministic/content-addressed mode),
- rendering outputs and trace maps must match (in deterministic mode),
- verification/activation decisions must match.

See: `../01-principles/03-determinism.md`

---

## What must be pinned?
Correctness-critical references must be pinned:
- mandate version
- blueprint version
- policy versions
- exchange_id / pack_id (no floating “latest” in execution/render correctness paths)
- templates and renderer versions for render reproduction

---

## How do we prove the system is “tight”?
By conformance:
- schema validation for artifacts,
- determinism tests for gates/build/render,
- fail-closed verification tests,
- trace coverage tests for rendering,
- “no canon mutation” tests for all modules,
- end-to-end auditability and correlation across telemetry.

See: `../07-implementation-guides/03-testing-conformance.md`

---

## Where do architecture changes get recorded?
In ADRs:
- any change to invariants,
- artifact schema changes,
- gate behavior changes,
- compatibility policy changes.

See: `adr/`

---

## What is the minimal reading path for engineers?
1. `../01-principles/01-invariants.md`
2. `../02-architecture/02-lifecycle.md`
3. `../04-artifacts-contracts/01-artifact-catalog.md`
4. `../05-protocols-flows/02-execution-path.md`
5. `../05-protocols-flows/04-feedback-path.md`
6. Node docs relevant to your integration under `../03-nodes/`
