# Determinism (kOA system)

**Status:** Normative (kOA)  
**Scope:** End-to-end behavior across Orgo → Kristal → Konnaxion → Architect, including gates, builds, distribution, and rendering.

## 1) Purpose

Determinism is the property that the system produces **identical** (or **canonically identical**) outputs when given identical pinned inputs + pinned configuration + pinned policies, and that failures occur with **stable error behavior**.

This document defines:
- what kOA means by determinism,
- what must be pinned/recorded,
- what classes of nondeterminism are forbidden at boundaries,
- what conformance tests kOA requires.

**Not in scope:** re-specifying Kristal artifact schemas, hash targets, canonicalization, signing formats, or Kristal compiler internals. Those are treated as external normative dependencies (see `../40-integration/kristal-v4/contract-pointers.md`).

## 2) Definitions

- **Deterministic output:** bit-identical outputs for the same pinned inputs/config/policies.
- **Canonically identical output:** outputs may differ in non-semantic representation but are identical after applying the declared canonicalization (e.g., JCS).
- **Pinned inputs:** a content-addressed snapshot of all upstream artifacts used (inputs, Claim-IR, Resolved Claim-IR, rulesets, etc.).
- **Pinned configuration:** exact compiler/runtime config captured by a config hash and versioned component identities.
- **Portable policy:** an enumerated policy selection recorded in manifests/records such that another implementation can reproduce behavior.

## 3) Determinism layers (kOA)

kOA determinism is enforced at multiple layers:

### 3.1 Workflow determinism (Orgo)
- Stage ordering is fixed and auditable.
- Validation failure deterministically blocks compilation (“no compile on fail”).
- A Build Record deterministically binds inputs → config → policy → outputs.

### 3.2 Canon determinism (Kristal compilation)
- Exchange and Runtime Pack are built deterministically under pinned inputs/config/policies.
- Content-addressed IDs, hashes, and manifests must be reproducible across implementations.

### 3.3 Distribution determinism (Konnaxion)
- Activation is atomic and verification is fail-closed.
- Rollback behavior is deterministic given the same trigger sequence and available pack set.

### 3.4 Rendering determinism (Architect-Render)
- Rendering must be deterministic from the same validated inputs + templates + parameters.
- Rendering must not introduce new facts; it must trace every factual statement or refuse/omit deterministically.

## 4) What MUST be pinned/recorded (minimum reproducibility surface)

kOA requires that the following are recorded in the operational evidence (Build/Release records and/or referenced manifests):

1) **Inputs**
- input snapshot identifier(s)
- upstream artifact references required to rebuild

2) **Policy selections**
- policy IDs/versions (and any enumerated policy knobs that affect output)

3) **Component identities**
- compiler / validator / renderer / packager names + versions
- dependency lock versions where applicable

4) **Configuration hash**
- hash of the effective configuration that influences outputs

5) **Canonicalization declaration**
- declared canonicalization profile + version for any content-addressed JSON material

6) **Integrity declarations**
- declared file hashes and signatures (if present) sufficient for offline verification

## 5) Forbidden (or constrained) sources of nondeterminism

### 5.1 Wall-clock timestamps
- Timestamps may exist for audit and correlation, but MUST NOT affect any content-addressed ID or hash target unless a profile explicitly includes them.

### 5.2 Randomness / probabilistic components
- Any probabilistic component MUST support a deterministic mode for tests and reproducible builds:
  - fixed seed
  - pinned model version
  - pinned dependency versions
  - frozen candidate sources (if applicable)
- If deterministic mode is impossible, the nondeterministic stage must be moved **upstream of the truth boundary** and its outputs must be frozen as inputs (e.g., store Claim-IR and do not re-extract).

### 5.3 Concurrency and ordering
- Any output that depends on iteration order MUST define deterministic sorting and tie-breakers.
- Parallelism may be used, but output order and aggregation MUST be deterministic.

### 5.4 Floating-point / platform differences
- Algorithms that can vary across CPU/OS (float math, hashing libraries, locale collation) MUST be stabilized by:
  - integer-only representations where possible,
  - explicit rounding rules,
  - canonical byte encodings,
  - pinned library implementations.

### 5.5 External dependencies (network, live services)
- Any external dependency MUST be represented as a content-addressed input snapshot (or otherwise pinned and recorded) for any deterministic build claim.
- Network lookups are not allowed inside deterministic compilation unless their results are captured as pinned inputs.

## 6) Canonicalization and hashing (kOA rule, Kristal normative)

kOA requires that all content-addressed JSON material uses the declared canonicalization profile/version and that the system’s reproducibility tests compute IDs/hashes using that canonicalization.

The **normative** canonicalization/hashing rules for Kristal artifacts are treated as external dependencies and must be followed exactly as pinned in `../40-integration/kristal-v4/contract-pointers.md`.

## 7) Deterministic failure behavior

Determinism also applies to failures:

- Given identical inputs/config/policies, validation failures must produce stable error categories/codes and stable gating outcomes.
- Rendering must deterministically:
  - trace, or
  - omit, or
  - mark uncertain (only when upstream uncertainty exists), or
  - refuse,
  under the same policy + inputs.

## 8) Conformance expectations (what kOA will test)

kOA conformance includes (minimum):
- Canonicalization and hashing checks under the declared profile.
- Rebuild determinism: same pinned inputs/config/policies → same content IDs and declared file hashes.
- Fail-closed verification: any declared integrity mismatch blocks publish/activation.
- Deterministic rendering: same validated inputs/template/params → same outputs and trace coverage (or same deterministic refusal/omission behavior).

## 9) Operational notes

- Determinism does not prohibit optimization; it constrains optimization to **declared policies** and requires those policy selections to be recorded.
- Where kOA must interoperate with multiple implementations, kOA should treat “deterministic build rules” and “reproducibility acceptance tests” from the pinned Kristal dependency as gating requirements for release.