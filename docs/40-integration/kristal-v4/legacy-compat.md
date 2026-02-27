# Legacy Compatibility (kOA ↔ Kristal v4)

## Purpose
Define how kOA handles legacy or out-of-date Kristal artifacts and related metadata **without** weakening the fail-closed truth boundary.

This document is **kOA-specific**. Normative Kristal contract details remain in the pinned Kristal v4 source.

---

## Compatibility stance

### 1) Input tolerance vs output strictness (normative)
kOA components MAY accept legacy spellings/fields **only as input**, but MUST:
- normalize to the pinned Kristal v4 canonical forms internally, and
- emit only canonical Kristal v4–conformant artifacts at boundaries, and
- record a normalization note in kOA-native operational artifacts (Build Record / Release Record / Konnaxion State) when legacy input was encountered.

### 2) Fail-closed on ambiguity (normative)
If legacy input prevents unambiguous normalization (unknown profile, missing required fields, unclear signature target), kOA MUST fail-closed:
- Orgo blocks compilation/release steps.
- Konnaxion blocks activation.
- The failure reason MUST be recorded with a stable reason code.

---

## Allowed legacy forms (input only)

### A) Canonicalization identifiers
kOA MAY accept legacy canonicalization identifiers ONLY if they can be deterministically mapped to the pinned Kristal v4 canonical identifier set.

Examples of legacy spellings that may appear:
- `canonicalization_profile: "jcs-rfc8785"`
- `canonicalization_version: "1.0"`
- combined forms such as `jcs-rfc8785@1.0`

Normalization behavior:
- map to the pinned Kristal v4 canonical values for profile/version (see `contract-pointers.md`).

If the mapping is not explicitly defined for the pinned Kristal version, treat as **unsupported** and fail-closed.

### B) Legacy manifest field names (kOA operational ingestion only)
kOA MAY accept legacy field names in upstream operational records (e.g., build logs or older Orgo records) when a deterministic mapping exists, for example:
- `build_timestamp` → `created_at`
- `input_snapshots` → the canonical “inputs.*” structure (where applicable)
- `policy_selections` → the canonical “policies.*” structure

Rules:
- Only normalize when the mapping is one-to-one or losslessly representable.
- If normalization would discard meaning (lossy), mark the record as `DEGRADED_COMPAT` and require operator action.

### C) Key identifier naming
Legacy signature metadata may use `kid` in some ecosystems.
kOA policy:
- Accept `kid` only if it can be mapped deterministically to the pinned Kristal v4 `key_id` semantics.
- Never allow both `kid` and `key_id` to conflict; if both exist and differ, fail-closed.

---

## Compatibility levels

### Level 0 — Strict v4 only (recommended default)
- Reject any legacy spellings/fields.
- Best for production truth pipelines with controlled producers.

### Level 1 — Normalize legacy identifiers (recommended for migration windows)
- Accept legacy canonicalization identifiers and deterministic field aliases.
- Emit canonical v4 only.
- Require telemetry + audit notes when normalization occurs.

### Level 2 — Extended legacy acceptance (not recommended)
- Accept broader legacy variations.
- Higher risk of ambiguity and inconsistent verification.
- Only allowed with explicit operator approval and documented ADR.

kOA deployments MUST declare which level applies per environment (dev/stage/prod).

---

## Operational requirements

### Logging (normative)
When legacy normalization occurs, log:
- source artifact ref / producer
- legacy fields encountered
- canonical replacements applied
- whether the change was lossless
- reason code and compatibility level applied

### Metrics (recommended)
Track:
- count of legacy artifacts observed
- normalization success/failure rates
- top legacy patterns encountered
- time-to-zero legacy usage (migration progress)

---

## Security implications

Legacy acceptance expands the input surface. Therefore:
- only allow legacy normalization in controlled ingestion paths
- never bypass signature or hash verification due to legacy fields
- treat unknown legacy forms as potential tampering until proven otherwise

---

## Change control
- Any expansion of “Allowed legacy forms” requires an ADR under `docs/90-reference/adr/`.
- Compatibility level defaults and environment policy must be recorded in `koa-profile.md`.