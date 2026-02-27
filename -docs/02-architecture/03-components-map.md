# Components Map

**File:** `docs/02-architecture/03-components-map.md`  
**Status:** Canonical (descriptive; contracts are normative elsewhere)  
**Purpose:** Provide a system-level map of the ecosystem’s components, responsibilities, and interfaces—without re-defining detailed node specs or artifact schemas.

---

## 1) System at a glance

This ecosystem is organized as:

- **Control Plane:** Orgo (governance + workflow orchestration + audit)
- **Truth Plane (Canonical):** Kristal (Exchange + Runtime Pack; IDs, hashing, signatures; query contract)
- **Resolution Plane:** SenTient (reconciliation / normalization / ambiguity structures)
- **Articulation Plane:** Architect (Strategy + Render; deterministic outputs; trace)
- **Execution Plane:** SwarmCraft (tool/agent/human execution; telemetry)
- **Distribution + Interface Plane:** Konnaxion (pack distribution, verification, caching, activation/rollback; user-facing navigation)
- **Trust + Impact Plane:** EkoH (reputation/ethics, votes, impact ledger; never mutates canonical truth)

---

## 2) Component responsibilities (what each owns)

### Orgo (Control Plane)
**Owns**
- Workflow ordering and gating (ingest → Claim-IR → resolution → validation → compile → publish)
- Governance: approvals, policies, tenant boundaries, audit logging
- Build records: inputs, config hashes, policy selections, outputs (content-addressed IDs)
- Distribution triggers and distribution status tracking

**Does not own**
- Canon mutation from feedback
- Resolution internals
- Rendering internals

**Interfaces**
- Ingest adapters (sources → snapshots)
- Extractors (snapshots → Claim-IR)
- SenTient (Claim-IR → Resolved Claim-IR)
- Validation engine (Resolved Claim-IR → Validation Report)
- Kristal compiler (validated inputs → Exchange + Runtime Pack)
- Konnaxion (publish/activate packs; telemetry back)

---

### Kristal (Truth Plane / Canon)
**Owns**
- Canonical truth representation (Exchange)
- Offline execution representation (Runtime Pack)
- Canonicalization + hashing rules, signatures/trust roots
- Profiles/exports and query contract (portable semantics)

**Does not own**
- Workflow governance (Orgo)
- UI presentation (Konnaxion)
- Non-deterministic invention

**Interfaces**
- Compiler API (inputs → Exchange/Pack + manifests)
- Verification API (hash/signature verification)
- Query API (portable query semantics over Exchange/Pack)
- Integration contracts for Orgo / SenTient / Architect / Konnaxion

---

### SenTient (Resolution Plane)
**Owns**
- Deterministic resolution boundary:
  - candidate IDs (QIDs/PIDs), normalized literals
  - explicit ambiguity objects
  - structured warnings/errors
- Schema-valid, reproducible Resolved Claim-IR output

**Does not own**
- Canon compilation
- Validation acceptance policy
- Direct publication

**Interfaces**
- Input: Claim-IR batches (+ optional context)
- Output: Resolved Claim-IR + warnings/errors
- Optional: provenance attachments for audit

---

### Validation Engine (Acceptance Gate)
**Owns**
- Deterministic acceptance decision (“no compile on fail”)
- Validation report artifacts (machine-readable pass/fail + reasons)

**Interfaces**
- Input: Resolved Claim-IR + policy selections
- Output: Validation Report

---

### Architect (Articulation Plane)
Architect is explicitly split into two roles:

#### Architect-Strategy
**Owns**
- Planning and decomposition of objectives into structured work (Cases/Tasks)
- Selection of rendering specs/templates (but not editorial invention of facts)

**Interfaces**
- Input: Mandate + user goal + available canon/pack index
- Output: Plans → Orgo Cases/Tasks; Rendering Specs

#### Architect-Render
**Owns**
- Deterministic rendering from validated knowledge only
- Output packaging (Render Bundle: content + trace map + metadata)
- Refusal/error behavior when trace coverage is incomplete

**Interfaces**
- Input: Runtime Pack query results OR Exchange-derived verified query results + Rendering Spec
- Output: Render Bundle (text/other outputs + trace map)

---

### SwarmCraft (Execution Plane)
**Owns**
- Execution of typed work (tasks) via tools/agents/humans
- Telemetry emission (status, logs, correlation IDs, outputs)
- Retry and resilience at execution level (not truth mutation)

**Does not own**
- Canon creation
- UI distribution

**Interfaces**
- Input: Orgo Tasks (typed, scoped, policy-aware)
- Output: execution results + telemetry signals (which may open new governed work)

---

### Konnaxion (Distribution + Interface Plane)
Konnaxion is treated as two facets:

#### Distribution facet (Runtime Packs)
**Owns**
- Pack distribution (channels/tenants/environments)
- Integrity verification (fail-closed where declared)
- Caching/storage layout, activation, rollback/downgrade behavior
- Surfacing pack provenance/version metadata to higher layers

**Interfaces**
- Input: Published Runtime Packs + manifests + trust roots/keys
- Output: Activated pack set + activation state + telemetry

#### Interface / Navigation facet
**Owns**
- User navigation/search across available packs/exchanges
- Presentation routing to Architect-Render (render requests)
- Feedback capture (signals → Orgo as governed work)

---

### EkoH (Trust + Impact Plane)
**Owns**
- Trust/impact ledger: expertise/ethics scoring, vote aggregation, history
- Privacy and auditability for score changes
- Influence on prioritization/weighting (without mutating canon)

**Interfaces**
- Input: events/feedback/participation signals (from Konnaxion, Orgo, SwarmCraft)
- Output: trust metrics, vote results, impact reports

---

## 3) Cross-cutting subsystems (shared concerns)

### Identity, keys, and trust roots
- Tenant-scoped trust roots (verification must be tenant-correct)
- Key rotation and revocation policy
- Signing/verification utilities used by compiler, Orgo, Konnaxion

### Storage
- Immutable artifact storage (Exchange, manifests, build records)
- Pack storage (cache + activation states)
- Telemetry storage (logs, correlation IDs, metrics)

### Observability
- Correlation IDs across Orgo ↔ compiler ↔ distribution ↔ render ↔ execution
- Deterministic error codes for gates/refusals
- Reproducibility logs for rebuild verification

---

## 4) Interfaces summary (boundary artifacts)

This section is intentionally brief; schemas live in `docs/04-artifacts-contracts/`.

- **Claim-IR**: extractor → SenTient
- **Resolved Claim-IR**: SenTient → validation
- **Validation Report**: validation → Orgo/compile gate
- **Kristal Exchange**: compiler output (canonical truth)
- **Runtime Pack**: compiler output (offline execution package)
- **Render Bundle**: Architect-Render output (content + trace map)
- **Case/Task objects**: Orgo work model for all governed work
- **Telemetry events**: SwarmCraft/Konnaxion → Orgo/EkoH

---

## 5) Feedback policy (where “signals” go)

All feedback is treated as *operational signal*:
- It may open new Cases/Tasks (governed work)
- It may influence prioritization/trust/UX
- It must not mutate Exchange in-place

---

## 6) Minimal resonance note (non-normative)

The ecosystem can be described via symbolic ontologies (node semantics and cyclic rigor), but the operational source-of-truth is defined by contracts: typed artifacts, deterministic gates, and global invariants.

See:
- `docs/01-principles/01-invariants.md`
- `docs/02-architecture/02-lifecycle.md`
- `docs/04-artifacts-contracts/01-artifact-catalog.md`
