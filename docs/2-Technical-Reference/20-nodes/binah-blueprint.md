# Binah — Blueprint

**Normative for kOA:** YES
**External normative reference:** Kristal v4 (pinned)

Binah is the planning node that converts a mandate + available inputs into a **Blueprint**: an explicit, auditable plan for what will be built, how it will be grounded, and which downstream nodes will execute each step. Binah does not validate truth; it structures work so that truth validation is possible and deterministic.

## Responsibility

Binah owns:

* Producing a **Blueprint** that is complete enough for deterministic execution downstream
* Declaring required inputs, retrieval/grounding strategy, and policy gates to be applied
* Assigning execution responsibilities across downstream nodes (SenTient, Architect, Compiler, Daat)

Binah does not own:

* Ground-truth resolution (SenTient)
* Kristal artifact verification or publication (Daat)
* Distribution/activation (Konnaxion)
* Governance approvals (Orgo), except to request them via work items

## Inputs

* **Mandate Bundle** (kOA artifact) — goals, constraints, required channels/environments, approvals state
* **Input Snapshot References** (from Chokmah) — opaque references to deterministic input sets
* **Policy Configuration** (kOA profile) — which policy families must be enforced downstream
* **System Capabilities** — supported compilers/runtimes, target environment matrix, feature flags

## Outputs

* **Blueprint** (kOA artifact)

  * Plan steps with explicit stage boundaries
  * Declared evidence/anchor requirements per step
  * Declared policy gates (by name), not their internal Kristal details
  * Expected Kristal artifact types to be produced (by type only)
* **Orgo Work Items** (optional)

  * Requests for missing inputs, approvals, or exception handling
* **Diagnostics**

  * Reasons for blocked planning (missing inputs, conflicting constraints)

## Blueprint structure (kOA-level)

A Blueprint is an ordered set of steps. Each step must include:

* `step_id` (stable)
* `owner_node` (which node executes it)
* `inputs` (artifact references by type + ID)
* `outputs` (artifact references by type)
* `guards` (preconditions; including “must be Kristal-verified” where applicable)
* `policy_gates` (named gates to be enforced by downstream nodes)
* `determinism_constraints` (e.g., pinned toolchain required, canonicalization required)
* `rollback_plan` (required when the plan targets activation channels)

Binah may include optional metadata for scheduling or parallelization, but it must not introduce nondeterministic dependencies.

## Guards

Binah must refuse to emit a Blueprint if any of the following are true:

* Mandate is missing required approvals for the requested channel(s)
* Required input snapshot references are missing or not pinned
* The plan would require producing Kristal artifacts without a declared path through Daat
* The plan includes steps that depend on non-pinned tools, non-versioned policies, or “live” data without snapshot identity
* Target environments include incompatible runtime constraints without a mitigation/rollback plan

## Failure modes

* **F1 Input acquisition failures:** missing snapshot references, input identity ambiguity
* **F2 Planning failures:** inconsistent constraints, incomplete step guards, missing rollback plan
* **F4 Determinism failures (prevention):** plan depends on unpinned toolchain or nondeterministic execution
* **F6 Activation risk:** plan targets rollout without rollback path

(See `10-system/failure-modes.md`.)

## Operational notes

* **Idempotent planning:** If Mandate + Snapshot References + Profile are unchanged, Binah must emit the same Blueprint (or a blueprint with a stable semantic hash).
* **Change control:** Any change in inputs, policies, or target matrix requires a new Blueprint revision.
* **Traceability:** Blueprint step outputs must be traceable to their inputs via references captured downstream (Build/Release records).

## Interfaces with other nodes

* **Keter → Binah:** mandate content and governance constraints
* **Chokmah → Binah:** snapshot references for deterministic inputs
* **Binah → SenTient:** resolution steps and evidence requirements
* **Binah → Architect:** render strategy and deterministic constraints
* **Binah → Compiler:** toolchain pinning and packaging targets
* **Binah → Daat:** expected Kristal artifacts and publish/verification intent
* **Binah → Orgo:** approvals and remediation work items
