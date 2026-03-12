# Components

**Normative for kOA:** NO (system map; descriptive)  
**External normative references:** Kristal v4 (pinned) for Kristal artifact formats/schemas (referenced, not restated)

**Purpose:** Map the ecosystem’s components, responsibilities, and interfaces—without re-defining node specs or artifact schemas.

---

## 1) Planes (system at a glance)

- **Control Plane:** Orgo (governance + workflow orchestration + audit)
- **Truth Plane (Canonical):** Kristal (Exchange + Runtime Pack; canonical IDs, hashing, signatures; query contract)
- **Resolution Plane:** SenTient (reconciliation / normalization / ambiguity structures)
- **Articulation Plane:** Architect (Strategy + Render; deterministic outputs; trace)
- **Execution Plane:** SwarmCraft (tool/agent/human execution; telemetry)
- **Distribution + Runtime Plane:** Konnaxion (pack distribution, verification, caching, activation/rollback) + Malkuth (offline serving/execution substrate)
- **Trust + Impact Plane:** EkoH (reputation/ethics, votes, impact ledger; never mutates canonical truth)

---

## 2) Component responsibilities and interfaces

### 2.1 Orgo (Control Plane)

**Owns**
- Workflow ordering and gating (ingest → Claim-IR → resolution → validation → compile → publish)
- Governance: approvals, policies, tenant boundaries, audit logging
- Build records: inputs, config hashes, policy selections, outputs (content-addressed references)
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
- Konnaxion (publish/fetch/verify/activate packs; telemetry back)
- EkoH (optional governance-weighting signals)

---

### 2.2 Kristal (Truth Plane / Canon)

**Owns (external normative)**
- Canonical truth representation (Exchange)
- Offline execution representation (Runtime Pack)
- Canonicalization + hashing rules, signatures/trust roots
- Profiles/exports and query contract (portable semantics)

**Does not own**
- Workflow governance (Orgo)
- UI presentation (Konnaxion)
- Downstream non-deterministic invention

**Interfaces (as used by kOA)**
- Compiler API (inputs → Exchange/Pack + manifests)
- Verification API (hash/signature verification)
- Query API (portable query semantics over Exchange/Pack)
- Integration contracts for Orgo / SenTient / Architect / Konnaxion

> kOA does not restate Kristal contracts here. See `docs/40-integration/kristal-v4/`.

---

### 2.3 SenTient (Resolution Plane)

**Owns**
- Deterministic resolution boundary (candidate IDs, normalized literals, explicit ambiguity objects, structured warnings/errors)
- Schema-valid, reproducible Resolved Claim-IR output (Kristal-defined artifact)

**Does not own**
- Canon compilation
- Validation acceptance policy
- Direct publication

**Interfaces**
- Input: Claim-IR batches (+ optional context)
- Output: Resolved Claim-IR + warnings/errors
- Optional: provenance attachments for audit

---

### 2.4 Validation Engine (Acceptance Gate)

**Owns**
- Deterministic acceptance decision (“no compile on fail”)
- Validation report artifacts (machine-readable pass/fail + reasons; Kristal-defined artifact)

**Interfaces**
- Input: Resolved Claim-IR + policy selections
- Output: Validation Report

---

### 2.5 Architect (Articulation Plane)

Architect is explicitly split into two roles.

#### 2.5.1 Architect-Strategy
**Owns**
- Planning and decomposition of objectives into structured work (Cases/Tasks)
- Selection of rendering specs/templates (but not editorial invention of facts)

**Interfaces**
- Input: Mandate + user goal + available canon/pack index
- Output: Plans → Orgo Cases/Tasks; Rendering Specs

#### 2.5.2 Architect-Render
**Owns**
- Deterministic rendering from validated knowledge only
- Output packaging (Render Bundle: content + trace map + metadata; kOA-owned)
- Refusal/error behavior when trace coverage is incomplete

**Interfaces**
- Input: Runtime Pack query results (via Malkuth/Konnaxion) OR verified Exchange-derived query results + Rendering Spec
- Output: Render Bundle (content + trace map + events/metadata)

---

### 2.6 SwarmCraft (Execution Plane)

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

### 2.7 Konnaxion (Distribution + Interface Plane)

Konnaxion is treated as two facets.

#### 2.7.1 Distribution facet (Runtime Packs)
**Owns**
- Pack distribution (channels/tenants/environments)
- Integrity verification (fail-closed where declared)
- Caching/storage layout, activation, rollback/downgrade behavior
- Surfacing pack provenance/version metadata to higher layers

**Interfaces**
- Input: Published Runtime Packs + manifests + trust roots/keys
- Output: Activated pack set + activation state + telemetry

#### 2.7.2 Interface / Navigation facet
**Owns**
- User navigation/search across available packs/exchanges
- Presentation routing to Architect-Render (render requests)
- Feedback capture (signals → Orgo as governed work)

---

### 2.8 Malkuth (Runtime Substrate)

**Owns**
- Serving the active pack read-only (offline-first)
- Deterministic query/lookup over pack payloads
- Optional routine execution under sandbox/resource controls
- Runtime telemetry (without canon mutation)

**Interfaces**
- Input: Activated pack mount + activation metadata (from Konnaxion)
- Output: query/execution results + telemetry

---

### 2.9 EkoH (Trust + Impact Plane)

**Owns**
- Trust/impact ledger (expertise/ethics scoring, vote aggregation, impact history)

**Non-negotiable boundary**
- EkoH never mutates canonical truth; it produces governance-weighting signals.

---

## 3) Where the detailed contracts live

- Node-level responsibilities and boundary specifics: `docs/20-nodes/*` (one page per component role).
- kOA-owned artifact contracts and schemas (e.g., Orgo Case/Task, Build/Release records): `docs/30-artifacts/*`.
- Kristal artifact contracts/schemas (Exchange, Runtime Pack, Claim-IR, Resolved Claim-IR, Validation Report): referenced via the pinned Kristal v4 dependency (not duplicated here): `docs/40-integration/kristal-v4/`.