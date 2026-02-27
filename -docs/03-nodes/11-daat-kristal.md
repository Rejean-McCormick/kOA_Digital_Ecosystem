# Da’at (Truth Pivot / Canon) — Kristal

Kristal is the ecosystem’s truth pivot: the component that turns **validated resolved knowledge** into **canonical truth** (Kristal Exchange) and a **portable derived execution form** (Runtime Pack). It is the boundary where “proposals” become “canon.”

Kristal does not decide what passes validation (Orgo does), does not execute work (SwarmCraft does), and does not generate user-facing prose (Architect-Render does). Kristal compiles.

---

## 1) Scope and non-goals

### In scope
- Deterministic compilation of:
  - **Kristal Exchange** (canonical truth artifact)
  - **Runtime Pack** (derived, portable, offline-executable artifact)
- Content addressing / deterministic identity (IDs derived from content + pinned context)
- Manifests, signatures/hashes, and reproducible build metadata
- Canonical projection formats needed by downstream systems (Architect, SwarmCraft, Konnaxion)

### Out of scope
- Validation policy decisions (Orgo owns the pass/fail gate)
- Meaning resolution from ambiguous evidence (SenTient owns resolution)
- Rendering and narrative production (Architect-Render owns outputs)
- Distribution, activation, rollback (Konnaxion owns pack delivery and activation)

---

## 2) Responsibilities

## 2.1 Canon creation: Kristal Exchange
- Compile a canonical, immutable representation of validated truth.
- Preserve provenance and traceability to source inputs.
- Encode ambiguity explicitly when present in upstream resolved artifacts.
- Provide stable reference primitives (IDs) usable across languages/toolchains.

## 2.2 Derived execution artifact: Runtime Pack
- Compile a portable representation optimized for:
  - offline use
  - low-latency querying
  - deterministic rendering and execution
- Ensure runtime packs are strictly derived from canon + pinned configuration.
- Provide compatibility metadata for consumer runtimes.

## 2.3 Build identity and reproducibility
- Produce deterministic identifiers for Exchange and Pack (when configured for content addressing).
- Emit build manifests binding:
  - mandate + blueprint versions
  - input snapshot IDs
  - resolved claim IDs / validation report hashes
  - compiler version + config hashes
  - resulting exchange_id and pack_id

---

## 3) Interfaces and boundaries (contracts)

## 3.1 Upstream inputs
Kristal consumes only **validated** inputs. Minimum dependencies:
- `mandate_bundle` reference (pinned version)
- `blueprint_bundle` reference (pinned version)
- `resolved_claim_ir[]` (or equivalent resolved structured knowledge)
- `validation_report` (MUST be PASS)
- compiler configuration (pinned) and schema versions

## 3.2 Downstream consumers
- **Orgo**: receives build outputs and manifests for audit, release, and governance.
- **Konnaxion**: receives Runtime Pack bundles for distribution/activation.
- **Architect-Render**: reads Exchange or Runtime Pack for deterministic rendering + trace.
- **SwarmCraft**: reads Runtime Packs as execution substrates (indexes, dictionaries, task assets).

## 3.3 Hard boundary rules
- Kristal MUST NOT compile if validation fails (**NO COMPILE ON FAIL**).
- Kristal Exchange MUST be immutable once published (no in-place mutation).
- Runtime Packs MUST be derivable from Exchange + pinned configuration (no hidden inputs).
- Kristal MUST NOT invent facts; it only compiles from validated resolved inputs.

---

## 4) Core artifacts

## 4.1 Kristal Exchange (canon)
The Kristal Exchange is the authoritative truth artifact.

Recommended contents:
- `exchange_manifest.json` (mandatory)
- canonical knowledge payload:
  - entities / concepts
  - relations / edges
  - claims (structured)
  - provenance pointers
  - ambiguity markers where present
- optional indexes for fast lookup (still part of canon if included)

Recommended properties:
- content-addressed identity (`exchange_id`)
- deterministic ordering / canonical serialization
- explicit schema versioning

## 4.2 Runtime Pack (derived)
A Runtime Pack is an offline-executable projection of canon.

Recommended contents:
- `pack_manifest.json` (mandatory)
- query indices / tables / dictionaries
- projection-specific assets (templates, locale bundles, embeddings/indexes if applicable)
- optional signatures/hashes

Recommended properties:
- `pack_id` (content-derived if feasible)
- compatibility fields (contract version(s), required projections)
- deterministic build reproducibility

## 4.3 Build Manifest (binding record)
Kristal MUST emit a build manifest that binds inputs to outputs.

Minimum fields:
- `build_id`
- `exchange_id`, `pack_id`
- pinned `mandate_id` + version
- pinned `blueprint_version`
- `input_snapshot_ids[]`
- `resolved_claim_ids[]` (or a hash root)
- `validation_report_hash`
- `compiler_version`, `compiler_config_hash`
- `schema_versions`
- timestamps and actor/service identity

---

## 5) Determinism and identity

Kristal supports two modes (choose one per ecosystem policy):

### 5.1 Deterministic / content-addressed mode (preferred)
- `exchange_id` and `pack_id` are derived from:
  - canonical serialized payload
  - pinned mandate/blueprint references
  - compiler config hash and schema versions
- Identical inputs + configuration MUST yield identical IDs and identical artifacts.

### 5.2 Stable-ID mode (acceptable if required)
- IDs are stable but not purely content-derived.
- Build manifest MUST still bind all inputs/versions/config hashes for replayability.

---

## 6) Gates (required validations inside Kristal)

Before producing Exchange/Pack, Kristal MUST verify:
1) `validation_report.status == PASS`
2) mandate + blueprint versions are pinned
3) schema versions are compatible
4) input payloads match schema (resolved claims, provenance pointers)
5) compiler config is pinned and recorded
6) canonical serialization rules are applied (stable ordering)

If any gate fails:
- do not publish Exchange or Pack
- emit a deterministic failure report (error codes + diagnostics)

---

## 7) Security and integrity

### 7.1 Integrity guarantees
- Manifests MUST include hashes for payload files (at least manifest hash; ideally file hashes).
- If signatures are used:
  - manifest signatures MUST be verifiable offline
  - key identifiers (kid) MUST be included
  - trust roots MUST be pinned and not fetched at activation time as a correctness dependency

### 7.2 Confidentiality and redaction (policy-dependent)
If the ecosystem supports private canon:
- Exchange may be split into:
  - public projection
  - private projection
- Manifests must declare confidentiality class and access policy tags.
Kristal should not decide access; it must label artifacts for downstream enforcement.

---

## 8) Observability (minimum)

Kristal MUST emit:
- `build_id`, `exchange_id`, `pack_id`
- pinned mandate and blueprint versions
- validation report hash and resolved claim root hash
- compiler version and config hash
- file counts/sizes (Exchange + Pack)
- durations per compilation step
- deterministic error codes on failure

Recommended metrics:
- compile success rate / failure causes
- artifact size trends
- time-to-compile by projection
- reproducibility checks (sampled rebuild comparisons)

---

## 9) Failure modes (expected behavior)

- Validation not PASS: refuse compilation (no artifacts).
- Schema mismatch: refuse compilation; emit diagnostics referencing violating fields.
- Non-deterministic serialization detected: refuse or fall back to deterministic enforcement, then rebuild.
- Missing provenance pointers: refuse compilation (traceability broken).
- Signature material missing (when required): refuse publication (fail closed).

---

## 10) Conformance tests (required)

A conformant Kristal implementation MUST provide tests for:
- **No compile on fail**: PASS required; FAIL blocks Exchange/Pack.
- Immutability: published Exchange cannot be mutated in-place.
- Determinism: identical inputs/config yield identical artifacts and IDs (in content-addressed mode).
- Manifest completeness: build manifest binds all inputs/versions/config hashes.
- Provenance integrity: every compiled claim maps to a resolved upstream claim with provenance.
- Offline verification: signatures/hashes verifiable without network access (if signing enabled).

---

## 11) Appendix: why Da’at is the pivot

Da’at is where the system “knows” in a strict sense: it is not the place of interpretation, debate, or execution. It is the place where governed validation is crystallized into canonical truth and portable runtime substance—so every downstream action remains traceable, reproducible, and fail-closed.
