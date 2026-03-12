# kOA Profile for Kristal v4 (Pinned)

**Normative for kOA:** YES  
**External normative references:** Kristal v4 (pinned) docs + schemas (Kristal-owned artifact contracts)

## Purpose
This document defines the **kOA-required profile** for producing, distributing, verifying, and consuming Kristal artifacts (Exchange + Runtime Packs) within the kOA ecosystem, **without duplicating** Kristal’s normative schemas/spec.

kOA treats Kristal v4 as the **sole normative source** for Kristal-owned artifact formats and schemas. This document adds only **kOA-specific enforcement** (gates, compatibility rules, activation behavior).

---

## 1) Scope and normative sources

### 1.1 Normative (external)
Kristal v4 pinned dependency (vendored/submodule) is the normative source for:

- Exchange Manifest schema: `vendor/kristal/02-schemas/exchange-manifest.schema.json`
- Runtime Pack Manifest schema: `vendor/kristal/02-schemas/runtime-pack-manifest.schema.json`
- Validation Report schema (if consumed as a contract): `vendor/kristal/02-schemas/validation-report.schema.json`
- Core spec (IDs/canonicalization/hashing, signatures/trust): `vendor/kristal/01-core-spec/`

### 1.2 Normative (kOA)
kOA requirements for:
- activation/rollback behavior at the edge (Konnaxion)
- compatibility checks and downgrade prevention
- operational conformance tests and enforcement gates

These are kOA-owned rules and must not redefine Kristal identity semantics.

---

## 2) kOA baseline conformance level

kOA’s baseline profile targets **Kristal core semantics** as defined by the pinned Kristal v4 dependency, including:
- RFC 8785 (JCS) canonicalization (per Kristal’s declared profile identifiers)
- SHA-256 content addressing for core hashes/IDs
- explicit determinism declaration for Runtime Packs (when the schema/profile requires it)

Optional capabilities (exports, advanced integrity, provenance packaging) MUST be claimed via explicit Kristal profiles and must not redefine core identity semantics.

---

## 3) Canonicalization, hashing, and ID rules (kOA-enforced)

### 3.1 Canonicalization identifiers
kOA producers MUST record canonicalization fields using the canonical values:
- `canonicalization_profile = "kristal.v3:jcs-rfc8785"`
- `canonicalization_version = "1"`

Legacy spellings MUST NOT appear in emitted artifacts or examples (including `"jcs-rfc8785"`, `"jcs-rfc8785@1"`, `"1.0"`).

### 3.2 Hash/sign workflow
kOA follows Kristal’s normative order:
1) remove signature material from the signing/hash target (as defined by Kristal)
2) canonicalize via the declared profile (JCS)
3) hash (SHA-256 for core IDs and declared integrity)
4) verify and/or attach signatures

### 3.3 Signature envelope placement
If signatures are present, they MUST be carried in a top-level `signatures[]` array (not interleaved into signed content).

### 3.4 Hash object field name
kOA uses `alg` (not `algo`) in hash objects.

---

## 4) Determinism and reproducibility (kOA baseline)

### 4.1 Determinism declaration (Runtime Packs)
If the pinned Kristal schema/profile requires an explicit determinism declaration, kOA Runtime Packs claiming baseline conformance MUST set it as required by that schema/profile (e.g., a boolean `build.deterministic = true` or equivalent field).

### 4.2 Inputs and config are part of determinism
A deterministic build MUST be parameterized by an explicit input snapshot set; the Runtime Pack Manifest MUST reference the Exchange it was compiled from; compiler configuration affecting output bytes MUST be hashed and recorded.

### 4.3 Portable policy surface
Build-affecting behaviors MUST be selected from the allowed policy set. If a needed behavior is not covered, the build must either:
- propose a new standard policy, or
- emit a non-baseline (non-core) profile pack.

Ordering policies affecting bytes MUST be recorded (e.g., data ordering / projection ordering / index ordering policies).

---

## 5) Integrity and fail-closed enforcement (kOA baseline)

### 5.1 Fail-closed semantics
If any integrity material is declared (hashes, signatures, signer identity/key reference), verifiers MUST fail closed on:
- verification failure,
- malformed integrity material,
- hash mismatch,
- missing required files referenced by the manifest (per policy).

### 5.2 kOA requirement: edge verification before activation
Konnaxion MUST verify a pack before activation, including:
- manifest parsing + schema validation (against pinned Kristal schema)
- integrity verification (manifest and/or file hashes, per policy)
- signature verification (if `signatures[]` present) using pinned trust roots
- required file presence checks

If verification fails: do not activate; remain on current/last-known-good; emit deterministic error/reason codes.

---

## 6) Compatibility, activation, downgrade prevention, rollback (kOA baseline)

### 6.1 Compatibility checks (mandatory)
Before activation, Konnaxion MUST validate compatibility between:
- the pack’s declared query/contract version (as defined by the pinned Kristal dependency), and
- the client runtime’s supported contract versions.

It MUST also check required capabilities/projections and relevant locale compatibility; on incompatibility: do not activate; remain on current pack; emit deterministic diagnostics.

### 6.2 Atomic activation
Activation MUST be atomic (no partial state).

### 6.3 Downgrade prevention
Default policy: do not activate a pack with lower `pack_version` than current; downgrade is permitted only via explicit rollback or operator-approved policy.

### 6.4 Deterministic rollback
Konnaxion MUST provide deterministic rollback to a verified pack:
- pinned known-good rollback
- last-known-good rollback

---

## 7) Query semantics (kOA baseline)

Runtime Packs MUST support constrained offline queries; baseline requires deterministic ordering and deterministic paging semantics as declared.

kOA pins the query contract version via the pinned Kristal dependency and enforces the compatibility checks in §6.1.

---

## 8) Rendering and “no new facts” downstream (kOA baseline)

### 8.1 Separation of Strategy vs Render
Rendering must be deterministic and must not introduce new facts; kOA keeps Architect split into Strategy vs Render to prevent truth drift.

### 8.2 Render Bundle requirement (system boundary)
Architect-Render produces a typed Render Bundle with mandatory `trace_map`; outputs must be reproducible and traceable to validated lineage artifacts.

Architect-Render MUST NOT introduce unsupported factual assertions; if trace coverage is insufficient it MUST omit, mark uncertain (only if uncertainty exists upstream), or refuse deterministically.

---

## 9) Required conformance gates (kOA)

kOA deployments MUST gate releases on:
- schema conformance for Exchange + Runtime Pack manifests (pinned Kristal schemas)
- determinism declaration when required by the pinned Kristal schema/profile
- fail-closed verification behavior at activation time
- compatibility checks and downgrade/rollback rules
- render determinism + no-new-facts + trace coverage enforcement for user-facing outputs