# Nodes

**Normative for kOA:** YES
**External normative reference:** Kristal v4 (pinned)

This section defines the kOA node model: each node has a clear responsibility, declared inputs/outputs (at the artifact-type level), and explicit failure modes. Node pages are **kOA interface specs**, not Kristal artifact specs.

Kristal artifacts (Exchange / Runtime Pack / Render Bundle and their schemas) are referenced via `40-integration/kristal-v4/` and must not be duplicated here.

## How to read a node spec

Each node page follows the same structure:

1. **Responsibility**

   * What the node owns (and what it does not own)

2. **Inputs**

   * Artifact types and upstream dependencies (no Kristal field-level contracts)

3. **Outputs**

   * Artifact types, events, and side effects

4. **Guards**

   * Preconditions and gates (including “must be Kristal-verified” where applicable)

5. **Failure modes**

   * Node-local failures and propagation expectations (see `10-system/failure-modes.md`)

6. **Operational notes**

   * Rollout/rollback, observability requirements, idempotency guarantees

## Node topology

High-level flow (conceptual):

* **Mandate** → **Inputs** → **Blueprint** → **Resolution** → **Verification** → **Compile/Pack** → **Distribute/Activate** → **Runtime**
* Truth boundary enforcement is applied wherever Kristal artifacts are consumed or produced.

## Node index

* `keter-mandate.md` — mandate definition, governance entrypoint
* `chokmah-inputs.md` — input acquisition and snapshotting
* `binah-blueprint.md` — blueprint formation and planning
* `tiferet-sentient.md` — resolution, grounding, and policy gating (content-level)
* `netzach-architect-strategy.md` — strategy orchestration (non-render)
* `hod-architect-render.md` — deterministic render planning (no new facts)
* `yesod-compiler.md` — compilation and packaging (deterministic toolchain)
* `daat-kristal-bridge.md` — Kristal boundary integration (verify/emit/publish)
* `chesed-konnaxion.md` — distribution, activation, rollback, channel management
* `gevurah-orgo.md` — workflow control, approvals, remediation tracking
* `malkuth-runtime.md` — runtime enforcement, safe modes, telemetry

## Common contracts (kOA-side only)

Nodes must satisfy these kOA-level requirements:

* **Explicit inputs/outputs:** no hidden dependencies
* **Idempotency:** re-running with identical declared inputs must be safe
* **Fail-closed behavior:** do not proceed when guards fail
* **Auditability:** emit sufficient records to reconstruct decisions and deployments
* **No silent mutation of canon:** feedback becomes governed work (Orgo), not implicit edits

## Kristal integration rule

If a node consumes or produces Kristal artifacts, the node page must:

* reference the pinned Kristal v4 dependency (`40-integration/kristal-v4/pinned-dependency.md`)
* reference the kOA profile (`40-integration/kristal-v4/koa-profile.md`)
* reference conformance requirements (`40-integration/kristal-v4/conformance.md`)

Node pages must **not** inline Kristal schemas or field lists.
