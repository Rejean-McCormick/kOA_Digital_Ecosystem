# ADR-0005: Compatibility & Versioning Policy (kOA)

**Status:** Accepted  
**Date:** 2026-02-27  
**Decision Owner:** Ecosystem Architecture  
**Scope:** kOA-native artifacts + integration compatibility; Kristal pinning strategy

---

## 1) Context

kOA integrates with Kristal as an external, normative artifact system. kOA also defines its own operational artifacts (Cases, Tasks, Build/Release Records, Konnaxion State) and must evolve them without breaking production pipelines.

Historical duplication of external contracts causes drift and ambiguity. Therefore, kOA must:
- pin external dependencies (Kristal) to a known version,
- define clear compatibility rules for kOA-native artifacts,
- treat ambiguous inputs as unsafe and fail-closed.

---

## 2) Decision

### 2.1 Kristal pinning (normative)
kOA MUST treat Kristal as an external dependency and MUST pin:
- a specific Kristal version (tag/commit)
- the exact schema set used for validation and generation

kOA documentation MUST reference Kristal via:
- `docs/40-integration/kristal-v4/pinned-dependency.md`
- `docs/40-integration/kristal-v4/contract-pointers.md`

kOA MUST NOT copy Kristal schemas as kOA-normative contracts.

### 2.2 kOA-native artifact versioning (normative)
All kOA-native artifacts MUST include:
- `schema_version` (semantic version, e.g., `1.2.0`)
- `created_at` timestamps
- stable IDs (UUID or policy-defined IDs)

Schema evolution rules:
- **PATCH** (`x.y.Z`): backward compatible bug fixes; no required-field additions.
- **MINOR** (`x.Y.z`): backward compatible additions (optional fields only).
- **MAJOR** (`X.y.z`): breaking changes (required-field changes, meaning changes, removals).

### 2.3 Compatibility guarantees (normative)
Within a given deployment environment:
- Producers MUST NOT emit artifacts newer than what consumers declare they support.
- Consumers MUST accept the full range of PATCH/MINOR versions within the same MAJOR line, unless explicitly prohibited by policy.

### 2.4 Deprecation policy (normative)
kOA MAY deprecate fields or behaviors, but MUST:
- keep deprecated fields readable for at least one MINOR cycle after deprecation is announced,
- emit warnings when deprecated inputs are used,
- remove deprecated fields only in the next MAJOR version.

### 2.5 Fail-closed on ambiguity (normative)
If an artifact cannot be validated against:
- a known schema version, or
- a pinned external contract (Kristal),
then kOA MUST fail-closed:
- Orgo blocks downstream pipeline steps
- Konnaxion blocks activation
- the failure is recorded with a stable reason code

---

## 3) Compatibility declarations

### 3.1 Producer declarations (recommended)
Producers SHOULD declare:
- supported schema versions they can emit
- maximum supported consumer versions (if constrained)

### 3.2 Consumer declarations (recommended)
Consumers SHOULD declare:
- accepted schema MAJOR lines
- optional feature flags for MINOR additions

---

## 4) Migration handling

### 4.1 Migration windows (recommended)
When introducing a breaking change:
- run dual-acceptance (old + new) in controlled environments
- emit canonical new versions
- convert legacy forms at boundaries only when deterministic and lossless

### 4.2 Legacy spellings for external contracts
Any tolerance for legacy spellings of Kristal-related fields must follow:
- `docs/40-integration/kristal-v4/legacy-compat.md`

---

## 5) Consequences

- External contract drift is prevented by pinning Kristal and referencing it directly.
- kOA-native artifacts evolve under semver rules with explicit compatibility boundaries.
- Operational safety is increased by failing closed on ambiguous inputs.

---

## 6) Follow-ups

- Ensure each kOA-native schema includes `schema_version`.
- Add conformance tests verifying version acceptance rules.
- Document supported versions per environment in ops runbooks.