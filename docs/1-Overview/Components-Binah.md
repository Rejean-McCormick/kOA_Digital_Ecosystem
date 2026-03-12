# Binah — Blueprint

**Normative for kOA:** YES  
**External normative reference:** Kristal v4 (pinned)

Binah is the planning node that turns a **Mandate** + **available inputs** into a **Blueprint**: an explicit, auditable plan describing what will be built, how it will be grounded, and which downstream nodes execute each step.

Binah does **not** validate truth. It structures work so that truth validation downstream is possible and deterministic.

---

## Responsibility

### Binah owns
- Producing a **Blueprint** that is complete enough for deterministic execution downstream
- Declaring required inputs, grounding strategy, and which policy gates must be enforced downstream
- Assigning execution responsibilities across downstream nodes (e.g., SenTient, Architect, Compiler, Daat)

### Binah does not own
- Ground-truth resolution (SenTient)
- Kristal artifact verification or publication (Daat)
- Distribution/activation (Konnaxion)
- Governance approvals (Orgo), except to request them via work items

---

## Inputs

- **Mandate Bundle** — goals, constraints, target channels/environments, approvals state
- **Input Snapshot References** — opaque references to deterministic input sets
- **Policy Configuration** — which policy families must be enforced downstream
- **System Capabilities** — supported compilers/runtimes, target environment matrix, feature flags

---

## Outputs

### Blueprint (kOA artifact)
A Blueprint specifies:
- Plan steps with explicit stage boundaries
- Evidence/anchor requirements per step
- Policy gates (by name)
- Expected Kristal artifact *types* to be produced (by type only)

### Optional: Orgo Work Items
Requests for missing inputs, approvals, or exception handling.

### Diagnostics
Reasons planning is blocked (e.g., missing inputs, conflicting constraints).

---

## Blueprint at a glance

A Blueprint is an **ordered set of steps**. Each step should make the following explicit (kOA-level, not schema-level):
- Stable step identity
- Which node executes the step
- Which artifact types are required as inputs and expected as outputs
- Preconditions/guards (including “must be Kristal-verified” where applicable)
- Named policy gates to be enforced
- Determinism constraints (e.g., pinned toolchain required)
- Rollback plan (required when the plan targets activation channels)

Binah may include scheduling/parallelization metadata, but must not introduce nondeterministic dependencies.

---

## Guards (when Binah refuses to emit a Blueprint)

Binah must refuse to emit a Blueprint if any of the following are true:
- Mandate is missing required approvals for the requested channel(s)
- Required input snapshot references are missing or not pinned
- The plan would require producing Kristal artifacts without a declared path through Daat
- The plan depends on non-pinned tools, non-versioned policies, or “live” data without snapshot identity
- Target environments include incompatible runtime constraints without a mitigation/rollback plan

---

## Failure modes

- **F1 Input acquisition failures:** missing snapshot references, input identity ambiguity
- **F2 Planning failures:** inconsistent constraints, incomplete step guards, missing rollback plan
- **F4 Determinism failures (prevention):** plan depends on unpinned toolchain or nondeterministic execution
- **F6 Activation risk:** plan targets rollout without rollback path

---

## Operational notes

- **Idempotent planning:** if Mandate + Snapshot References + Policy Profile are unchanged, Binah must emit the same Blueprint (or a blueprint with a stable semantic hash).
- **Change control:** any change in inputs, policies, or target matrix requires a new Blueprint revision.
- **Traceability:** step outputs must remain traceable to their inputs via references captured downstream (Build/Release records).

---

## Interfaces with other nodes

- **Keter → Binah:** mandate content and governance constraints
- **Chokmah → Binah:** snapshot references for deterministic inputs
- **Binah → SenTient:** resolution steps and evidence requirements
- **Binah → Architect:** render strategy and deterministic constraints
- **Binah → Compiler:** toolchain pinning and packaging targets
- **Binah → Daat:** expected Kristal artifacts and publish/verification intent
- **Binah → Orgo:** approvals and remediation work items

---

## Related pages

- Components: Orgo, Chokmah, SenTient, Daat, Konnaxion
- Artifacts: Mandate Bundle, Build Record, Release Record
- Operations: Pipeline, Releases, Rollbacks