# Kristal & Daat

Kristal is the system’s **truth compiler**. Daat is the **governed bridge** between kOA and Kristal that makes the Kristal interaction reproducible, auditable, and safe.

---

## What this component is responsible for

### Kristal (truth compiler)
- Compiles **validated knowledge** into **canonical truth artifacts**
- Produces derived runtime deliverables (e.g., **Runtime Packs**) that downstream runtime systems can verify and use offline
- Treats validation as a hard gate: **no compilation or publish of “truth” if validation failed**

### Daat (Kristal bridge)
- Pins and controls the **Kristal version** used (no floating “latest”)
- Ensures kOA only consumes/produces **Kristal-defined artifact types**
- Enforces **fail-closed** behavior at the boundary (schema/compatibility/integrity checks must pass)
- Records the operational evidence needed to reproduce and audit a build/release

---

## What it is not responsible for

- It does **not** decide what the product UI should look like (that’s rendering)
- It does **not** execute tasks or workflows (that’s execution)
- It does **not** mutate canonical truth directly via side effects (truth changes happen only through the governed pipeline)

---

## Inputs and outputs (high level)

### Typical inputs (to the Kristal boundary)
- Claim proposals (extracted facts that are not yet canonical)
- Resolution outputs (explicit decisions about entities/properties/values where needed)
- Validation policy context (what rules must be satisfied for truth to be accepted)

### Typical outputs (from the Kristal boundary)
- Validation report (pass/fail + reasons)
- Canonical truth artifacts (the compiled “truth” set)
- Runtime Pack + manifest (portable, verifiable bundle for runtime/offline use)

> Note: The exact schemas and names of Kristal artifacts are defined by the pinned Kristal version. This wiki intentionally avoids duplicating those contracts.

---

## How it works (conceptual)

1. **Pin Kristal**
   - Daat selects the approved Kristal version for the environment and build.

2. **Submit staged inputs**
   - kOA hands off prepared artifacts to the Kristal boundary (claims, resolutions, policy context).

3. **Validate (hard gate)**
   - Kristal produces a validation report.
   - If validation fails, the pipeline stops at the truth boundary.

4. **Compile**
   - When validation passes, Kristal compiles canonical truth.

5. **Package**
   - Kristal produces a Runtime Pack intended for verification and offline/runtime consumption.

6. **Handoff**
   - Daat registers outputs as immutable build results and makes them available to distribution/release.

---

## Operational expectations

- **Reproducibility**
  - The same pinned Kristal version + the same pinned inputs should produce the same canonical results (or canonically equivalent results).

- **Auditability**
  - Builds and releases should be able to reference “what Kristal produced” without copying it into operational records.

- **Safety at the boundary**
  - If any verification, compatibility, or integrity check fails, the system must not activate or promote outputs downstream.

---

## Common failure modes (non-technical)

- **Pinned version mismatch**
  - A build tries to compile with a Kristal version that is not approved for that environment.

- **Schema/contract mismatch**
  - An artifact presented to Kristal (or received from Kristal) does not conform to the expected contract.

- **Validation fails**
  - The report indicates a deterministic failure; compilation and release must not proceed.

- **Pack cannot be verified**
  - Downstream systems reject activation; rollback or hold is required.

---

## Upgrade and compatibility policy (practical)

- Treat Kristal upgrades like a release:
  - Pin a new version
  - Run conformance checks
  - Roll out gradually (channels/cohorts)
  - Keep rollback available to the last-known-good Kristal pin and pack lineage

---

## Quick FAQ

**Why have Daat instead of calling Kristal directly?**  
To keep the truth boundary governed: pinning, fail-closed checks, and operational evidence are enforced consistently.

**Why not document the Kristal schemas here?**  
To avoid drift. Kristal schemas are owned by Kristal and are referenced by pin, not duplicated in the wiki.

**Does Kristal run the product runtime?**  
No. Kristal produces truth and packs. Runtime systems consume verified packs to serve queries and experiences.