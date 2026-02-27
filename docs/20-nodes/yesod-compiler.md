# Yesod — Compiler (Build Executor)

**Normative for kOA:** YES  
**External normative reference:** Kristal v4 (pinned)

## Purpose
Yesod is the ecosystem’s **hermetic compilation executor**: it runs the pinned Kristal compiler in a controlled environment to produce **Kristal Exchange + Runtime Pack** outputs after Orgo authorizes compilation.

## Scope boundary
Yesod executes compilation. It does **not** decide acceptance (validation gate) and does **not** define Kristal artifact formats (Kristal is the normative source). Orgo orchestrates the workflow and invokes Yesod as a pipeline stage.

---

## Responsibilities

Yesod MUST:

1) **Execute compilation deterministically**
- Run the compiler with pinned inputs, pinned config, and pinned toolchain identity.
- Declare the enabled Kristal profiles that may affect outputs (if any).

2) **Produce schema-conformant Kristal artifacts**
- Exchange generation MUST satisfy the Kristal Exchange commit contract; the Exchange manifest MUST conform to the pinned Kristal schema.
- Runtime Pack generation MUST satisfy the Kristal Runtime Pack contract; the Pack manifest MUST conform to the pinned Kristal schema.

3) **Fail closed on declared integrity**
- If integrity material is declared (hashes/signatures/authority refs) and Yesod performs verification, it MUST fail closed on mismatch.

4) **Emit audit-grade build evidence**
- Emit stable output references/IDs back to Orgo for Build Record linkage (inputs → configs → outputs).
- Emit deterministic reason codes on failure.

Yesod MUST NOT:
- Compile when validation has not passed (it must reject compile requests missing Orgo authorization).
- Invent facts or alter validated content; it only compiles/packages under pinned inputs.

---

## Inputs

Yesod receives compilation jobs from Orgo that include:

- References to **Resolved Claim-IR** and a **PASS Validation Report** (or an Orgo-attested gate decision).
- Pinned **Mandate/Blueprint/Policy** references (or a single pinned context reference).
- Compiler identity/version + configuration reference (container digest / buildpack digest / binary hash).
- Declared Kristal profiles enabled for this compilation (if any).
- Target output locations (artifact store destinations / channels).

---

## Outputs

Yesod produces, at minimum:

- **Kristal Exchange** payload + **Exchange Manifest** (schema-conformant).
- **Runtime Pack** payload + **Runtime Pack Manifest** (schema-conformant).
- Optional export artifacts (only if implemented and declared as enabled profiles).

Yesod returns to Orgo:
- Artifact refs/IDs for Exchange + Pack + manifests
- Declared payload/file hashes
- Signature material (if produced)
- Execution environment digest pointer (for reproducibility/audit)

---

## Interfaces

### Orgo → Yesod (Compile Request)
A job request that includes:
- `build_id`
- gate authorization (attesting Validation PASS)
- input refs (Resolved Claim-IR, policy/blueprint/mandate refs)
- pinned compiler/config refs
- output destinations

### Yesod → Orgo (Compile Result)
A result object containing:
- compile status (success/fail + deterministic reason codes)
- output refs: exchange + pack + manifests
- integrity refs (hashes/signatures) where applicable
- reproducibility metadata pointer (execution environment digest)

---

## Invariants (non-negotiable)

1) **No compile without gate authorization**  
Yesod MUST reject compilation if not explicitly authorized by Orgo post-validation.

2) **Schema conformance (Kristal-owned)**  
Produced Exchange/Pack manifests MUST conform to the pinned Kristal schemas.

3) **Deterministic execution under pinned inputs**  
Given identical pinned inputs/config/toolchain, compilation MUST be replayable (subject to enabled profiles).

4) **Fail-closed integrity**  
If integrity is declared and verification is performed, mismatches MUST halt compilation and return a failure result (no partial canon).

---

## Failure modes

- **Gate missing/invalid** → reject compile request (authorization failure).
- **Schema validation failure (manifests)** → fail compile; emit deterministic error codes and pointers.
- **Integrity mismatch** (declared hash/signature) → fail closed.
- **Resource limit breach** (time/memory/disk) → fail with explicit resource-limit codes; do not emit partial canon.
- **Non-determinism detected** (byte drift across replay test) → fail build/conformance; quarantine the compiler/toolchain release.

---

## Observability

Yesod MUST emit trace spans/events that include:
- `tenant_id`, `environment`, `component`, `version`
- `build_id` and correlation id(s)
- output artifact IDs/refs (exchange_id/kristal_id, runtime_pack_id/pack_id, manifest refs)
- pinned policy/blueprint/mandate refs
- gate decision codes (as span events)
- deterministic failure reason codes on error

---

## Conformance expectations

- Provide valid/invalid fixtures for manifest schemas and ensure deterministic failure codes/locations.
- Ensure stage ordering is upheld end-to-end (Orgo enforces; Yesod refuses unauthorized compile).
- Ship and run the Kristal-required canonicalization/hash test vectors required by the pinned Kristal version.

---

## Pointers

- Orgo stage ordering and compile authorization: `50-operations/pipeline.md`
- kOA determinism requirements: `10-system/determinism.md`
- Kristal integration entry + conformance gates: `40-integration/kristal-v4/index.md`, `40-integration/kristal-v4/conformance.md`
- Pinned Kristal schemas (do not duplicate):  
  - `vendor/kristal/02-schemas/exchange-manifest.schema.json`  
  - `vendor/kristal/02-schemas/runtime-pack-manifest.schema.json`