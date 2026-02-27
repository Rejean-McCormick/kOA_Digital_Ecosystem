# Da’at: Kristal Bridge

**Role:** Boundary adapter between the kOA ecosystem and the Kristal truth system.
**Primary responsibility:** ensure kOA produces/consumes **Kristal v4–conformant artifacts** without re-specifying Kristal contracts inside kOA docs.

This node is the *integration surface*, not the Kristal spec.

## Responsibilities

* Pin the Kristal dependency version used by kOA (tag/commit) and keep it auditable.
* Route kOA pipeline stage outputs into Kristal compilation inputs (Claim-IR → Resolved Claim-IR → Validation results → compile request).
* Enforce that any produced Kristal artifacts (Exchange, Runtime Pack, their manifests) are **schema-valid** against pinned Kristal schemas.
* Maintain compatibility rules between kOA operational records (build/release records) and Kristal artifacts (IDs, references, provenance).
* Provide deterministic “artifact handoff” semantics:

  * what gets passed to Kristal
  * what comes back
  * what is recorded by Orgo

## Inputs

* Pre-truth artifacts from upstream stages:

  * Input Snapshot refs + provenance
  * Claim proposals (Claim-IR)
  * Resolved claims (Resolved Claim-IR)
  * Validation report (pass/fail + diagnostics)
* kOA policy bundles that influence compilation choices (as *parameters*, not schema changes)
* Pinned Kristal version reference (commit/tag) and schema locations

## Outputs

* Kristal canonical artifacts (produced by Kristal compiler):

  * Exchange artifact + manifest
  * Runtime Pack artifact + manifest
* kOA operational records updated with content-addressed references:

  * build record output refs
  * release record refs (if release proceeds)
* Compatibility/verification evidence:

  * schema validation results
  * canonicalization/profile checks
  * signer/trust assertions (if applicable)

## Invariants

* **Kristal is normative:** any field-level or schema-level truth about Exchange/Runtime Pack comes from the pinned Kristal v4 spec, not kOA docs.
* **No compile on fail:** if validation fails, compilation is not invoked and no new canonical artifacts are published.
* **Schema conformance:** the bridge must validate produced artifacts against pinned Kristal schemas before kOA considers them publishable/distributable.
* **Deterministic handoff:** given the same inputs + pinned Kristal version + pinned policies, the handoff request and recorded references are reproducible.
* **Stable referencing:** kOA records references to Kristal artifacts in a consistent, content-addressed way (no ad hoc IDs in place of canonical ones).

## Failure modes

* Kristal dependency not pinned / ambiguous version
* Schema mismatch between produced artifact and pinned schema
* Drift: kOA passes parameters not supported by the pinned Kristal compiler
* Non-determinism detected (toolchain/version drift)
* Partial publication (attempt to publish Exchange without corresponding recorded evidence)

## Observability

* Compile invocation counts, latency, and failure rate
* Schema validation pass/fail counts by artifact type
* “Pinned dependency drift” alerts (unexpected Kristal version changes)
* Determinism checks (same inputs → same artifact references) sampling
* Publication outcomes (published, rejected, quarantined)

## Interfaces

### Upstream

* Orgo (pipeline orchestrator; provides stage outputs and gating decisions)

### Downstream

* Kristal compiler toolchain (pinned version)
* Artifact store / registry (where Exchange and Packs are published)
* Konnaxion distribution (consumes Runtime Packs after verification)

## What lives elsewhere (by design)

* Normative Kristal artifact definitions and schemas: `docs/40-integration/kristal-v4/contract-pointers.md`
* kOA’s chosen profile / operational constraints: `docs/40-integration/kristal-v4/koa-profile.md`
* kOA conformance tests for Kristal artifacts: `docs/40-integration/kristal-v4/conformance.md`
