# Reading Path

This documentation set is organized around a single rule: **contracts and invariants define truth**. Read the docs in the order that matches your goal.

---

## Quick orientation (start here)

1. `index.md`  
2. `01-principles/01-invariants.md`  
3. `02-architecture/01-system-overview.md`  
4. `02-architecture/02-lifecycle.md`

If you read nothing else, read those four files.

---

## Track A — “I need the mental model” (architecture first)

1. `index.md`  
2. `02-architecture/01-system-overview.md`  
3. `02-architecture/03-components-map.md`  
4. `03-nodes/11-daat-kristal.md` (truth pivot)  
5. `03-nodes/05-gevurah-orgo.md` (control plane)  
6. `03-nodes/08-hod-architect-render.md` (deterministic outputs)  
7. `05-protocols-flows/` (all paths)

---

## Track B — “I’m implementing / integrating” (contracts first)

1. `01-principles/01-invariants.md`  
2. `04-artifacts-contracts/01-artifact-catalog.md`  
3. `04-artifacts-contracts/` (read the artifact specs you will produce/consume)
4. `05-protocols-flows/01-conscience-input-path.md`  
5. `05-protocols-flows/02-execution-path.md`  
6. `05-protocols-flows/03-anchoring-interface-path.md`  
7. `07-implementation-guides/02-integration-guide.md`  
8. `07-implementation-guides/03-testing-conformance.md`

---

## Track C — “I’m operating the system” (ops + failure modes)

1. `06-operations/01-build-pipeline.md`  
2. `06-operations/02-release-management.md`  
3. `06-operations/03-observability.md`  
4. `06-operations/04-incident-response.md`  
5. `05-protocols-flows/05-security-trust-path.md`  
6. `08-reference/adr/` (only the decisions relevant to your incident/change)

---

## Track D — “I need the node-by-node specification” (Sephiroth-style)

Read each node doc with the same template (Responsibilities → Interfaces → Artifacts → Invariants → Failure modes → Observability).

Recommended order:

1. `03-nodes/01-keter-mandate.md`  
2. `03-nodes/02-chokmah-inputs.md`  
3. `03-nodes/03-binah-blueprint.md`  
4. `03-nodes/06-tiferet-sentient.md`  
5. `03-nodes/05-gevurah-orgo.md`  
6. `03-nodes/11-daat-kristal.md`  
7. `03-nodes/07-netzach-architect-strategy.md`  
8. `03-nodes/08-hod-architect-render.md`  
9. `03-nodes/09-yesod-swarmcraft.md`  
10. `03-nodes/04-chesed-konnaxion.md`  
11. `03-nodes/10-malkuth-ekoh-ledger.md`

---

## How to use the spec (rules of interpretation)

- **Invariants win.** If any page conflicts with `01-principles/01-invariants.md`, treat that as a bug.
- **Artifacts are typed.** If a payload isn’t described in `04-artifacts-contracts/`, it is not a contract.
- **Paths are boundaries.** If a flow isn’t described in `05-protocols-flows/`, treat it as undefined behavior.
- **Changes require an ADR.** Any change to an invariant, artifact schema, or gate must be recorded in `08-reference/adr/`.

---

## Where the symbolic resonance is documented (minimal)

If you want the optional resonance note (not required for implementation):

- `02-architecture/04-parallel-note.md`
