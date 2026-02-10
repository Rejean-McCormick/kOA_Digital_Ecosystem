# Binah — Blueprint (Structural Design Node / “How”)

Binah is the **structure layer** of the ecosystem: it defines the schemas, ontologies, build configuration, and validation policies that make downstream processing deterministic and enforceable.

Binah is typically a **design-time configuration step** (not a runtime “reasoning” service). Once established, its outputs are treated as **canonical rules** for the project: downstream artifacts must conform or be flagged.

---

## 1) Responsibilities

### Binah MUST
- Define the **data models** used across the ecosystem (schemas for Claim-IR, Resolved Claim-IR, validation reports, manifests, envelopes).
- Define the **ontology / semantics surface**:
  - entity and relation types
  - property constraints
  - normalization rules (types, units, formats)
  - allowed ambiguity representation patterns
- Define the **build configuration** used by compilation and validation:
  - which modules are active
  - version pins
  - profile selections (core vs optional profiles)
  - determinism and reproducibility constraints
- Define **validation policies** and policy selections:
  - rule sets
  - deterministic error/warning codes
  - acceptance thresholds and “no compile on fail”
- Define **workflow intent** used by Orgo for routing, stage gating, and lifecycle constraints.
- Produce a single **versioned blueprint bundle** (see Section 3).

### Binah MUST NOT
- Mutate the Mandate (Keter). Binah shapes **how** to fulfill it, not **what** it is.
- Perform content processing (no extraction, no resolution, no compilation).
- Bypass governance: Binah changes must be introduced as explicit revisions and referenced by builds.

---

## 2) Inputs

Binah consumes:
- **Mandate / policy context** (Keter): scope, ethics, success criteria.
- **Raw input characteristics** (Chokmah): source types, formats, provenance requirements (not semantic truth).
- **Previous blueprint revisions** (if any): for compatibility analysis and controlled upgrades.
- **Operational constraints** (Orgo policy): environment, tenant rules, allowed profiles, signing requirements.

---

## 3) Outputs (canonical)

### 3.1 Binah Blueprint Bundle (recommended canonical artifact)

Binah SHOULD output a single bundle artifact (directory or archive) that is:
- **versioned**
- **deterministic** (stable ordering and canonical serialization)
- **hashable** (produces a `blueprint_hash` / `config_hash`)
- optionally **signed** (deployment policy)

Recommended contents:

- `blueprint_manifest.json`
  - `blueprint_id` (content-addressed or deterministic ID)
  - `blueprint_version` (semantic version)
  - `blueprint_hash` (hash of the full bundle)
  - `schema_versions` (declared schema versions for all typed artifacts)
  - `policy_selections` (portable policy identifiers; core vs profiles)
  - `module_activation` (what is enabled)
  - `compatibility` (min/max supported versions; migration notes)
- `schemas/`
  - JSON Schemas for typed artifacts and envelopes
- `ontology/`
  - entity/property definitions, mappings, constraints
- `validation/`
  - rule sets and deterministic code registry
- `pipeline/`
  - build and stage configuration defaults (for Orgo + compiler)
- `templates/` (optional; if treated as part of determinism surface)
  - render templates, language policies, formatting constraints

### 3.2 Direct consumers of the bundle

- **Orgo (Control Plane)** uses it to:
  - enforce stage order and gating
  - validate artifact schema versions
  - route work using declared taxonomy and policy constraints
- **SenTient (Resolution)** uses it to:
  - resolve against declared ontology, types, and ambiguity patterns
  - emit schema-valid Resolved Claim-IR
- **Kristal (Compilation)** uses it to:
  - compile Exchange/Runtime Packs under recorded policies
  - stamp manifests with config/policy hashes
- **Architect (Strategy/Render)** uses it to:
  - constrain planning/routing semantics (strategy)
  - enforce rendering constraints (render) without inventing facts
- **SwarmCraft (Execution)** uses it to:
  - interpret task envelopes and segmentation rules consistently

---

## 4) Interfaces (contract boundaries)

### 4.1 Upstream dependencies
- Keter → Binah: Mandate context constrains what structures/policies are allowed.
- Chokmah → Binah: Source types and provenance constraints inform schemas and pipeline settings.

### 4.2 Downstream boundaries
Binah is authoritative for:
- schema compatibility requirements
- ontology constraints and normalization rules
- policy selections and profile declarations
- module activation and version pins

Downstream components MUST refuse (or flag) inputs that claim to conform to Binah but violate:
- declared schemas
- declared ontology constraints
- declared determinism/reproducibility requirements
- declared policy selections

---

## 5) Invariants

Binah outputs are governed by the following invariants:

1. **Deterministic**: identical inputs and declared rules produce identical bundle hashes.
2. **Versioned**: every change is an explicit revision; no silent drift.
3. **Unambiguous**: no contradictory rules in the same bundle.
4. **Stability**: blueprint must not change on its own at runtime.
5. **Conformance authority**: downstream artifacts must conform or be flagged/rejected.
6. **Separation of concerns**: Binah defines structure; it does not process content.

---

## 6) Observability and telemetry

Binah SHOULD emit or enable:
- `blueprint_id`, `blueprint_version`, `blueprint_hash` for all builds and renders
- “coverage checks” (project completeness):
  - required fields present
  - required 5W coverage if applicable (policy-dependent)
- consistency checks:
  - no contradictory ontology rules
  - schema references resolve
  - policy selections are known and portable
- compatibility checks:
  - breaking-change detection vs previous revision
  - migration notes presence when breaking changes exist

---

## 7) Failure modes

### Configuration failures
- Missing required schema/policy declarations
- Contradictory ontology constraints
- Unknown policy selections or profile flags
- Unpinned or ambiguous version dependencies

### Downstream mismatch failures
- Orgo build references a blueprint that is not available/verified in the environment
- SenTient emits structures inconsistent with the declared schema/ontology
- Compiler cannot satisfy determinism requirements under the chosen options

Recommended handling:
- fail fast with deterministic codes and clear remediation instructions
- force explicit revision / migration workflow when breaking changes occur

---

## 8) Security considerations

- Blueprint bundles influence validation, compilation, and rendering behavior and SHOULD be treated as **sensitive configuration**.
- Deployments SHOULD:
  - store bundles in a controlled registry
  - require integrity verification (hash/signature) before use
  - pin bundles per tenant/environment
  - record the `blueprint_hash` into Orgo build records and Kristal manifests

---

## 9) Conformance tests (minimum)

An implementation claiming Binah conformance MUST provide tests for:
- **Determinism**: identical bundle inputs → identical `blueprint_hash`
- **Schema validity**: all declared schemas are valid and internally consistent
- **Policy portability**: policy selections use enumerated identifiers and are fully declared
- **Compatibility gating**: rejecting unknown/unsupported blueprint versions unless explicitly allowed
- **Downstream enforcement**: Orgo/SenTient/Compiler refuse artifacts that violate declared structure

---

## 10) Open specification items

- Formalize a canonical **Binah Blueprint Bundle schema** (`blueprint_manifest.json` + required directory structure).
- Define a stable **policy code registry** format and versioning rules.
- Define compatibility/migration semantics (e.g., “breaking change” taxonomy and required migration artifacts).
- Decide whether templates/language policies are inside the bundle (determinism surface) or referenced externally (policy-only).
