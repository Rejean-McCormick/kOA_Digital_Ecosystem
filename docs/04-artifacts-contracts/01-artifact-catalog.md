# Artifact Catalog

**Purpose:** Define the ecosystem’s authoritative artifacts (“typed outputs”), their ownership, identifiers, schemas, and invariants.  
**Normative:** This catalog is normative; an implementation is conformant only if it produces/consumes these artifacts according to the referenced contracts and gates.

---

## 1) Classification

### 1.1 Artifact classes
- **Pre-truth artifacts:** upstream proposals and checks (may be revised/rejected).
- **Canonical truth artifacts:** content-addressed truth (immutable once published).
- **Derived distribution artifacts:** portable packages derived from canonical truth.
- **Operational governance artifacts:** workflow/state/audit objects that control lifecycle and traceability.

### 1.2 Truth boundary (normative)
Canonical truth exists only **after** deterministic validation + compilation into **Kristal Exchange**. Validation failure MUST prevent compilation (“no compile on fail”).

---

## 2) Identity and versioning rules

### 2.1 Content-addressed identifiers
Orgo MUST record content-addressed references to stage outputs including (at minimum): Claim-IR, Resolved Claim-IR, Validation Report, Exchange (Exchange ID / hash), and Runtime Pack (Pack ID / hash).

### 2.2 Immutability rule
Kristal Exchange is immutable once compiled; changes produce new artifacts/versions (no in-place mutation).

### 2.3 Distribution integrity rule
Runtime Pack activation MUST verify declared hashes/signatures **fail-closed**; activation MUST be atomic; rollback MUST be deterministic.

---

## 3) Canonical artifact table

| Artifact | Class | Produced by | Consumed by | Primary IDs | Schema / format | Key invariants |
|---|---|---|---|---|---|---|
| Input Snapshots | Pre-truth | Orgo Ingest | Extractors, Orgo, Audit | `input_snapshots[]` (content refs) | snapshot manifest (**TBD**) | Immutable snapshots; reproducible input set recorded in Build Record |
| Claim-IR batch | Pre-truth | Extractors | SenTient, Validation | content ref | `schemas/claim-ir.schema.json` (**if present**) | Includes `claim_id`, evidence pointers; preserves uncertainty |
| Resolved Claim-IR batch | Pre-truth | SenTient | Validation, Kristal compile | content ref | `schemas/resolved-claim-ir.schema.json` (**if present**) | Ambiguity preserved; explicit resolution states; stable mapping to input `claim_id` |
| Validation Report | Pre-truth | Validation engine (Orgo-gated) | Operators, Orgo, Compiler gate | content ref | `schemas/validation-report.schema.json` (**if present**) | If validation fails: Orgo stops; surfaces report; no compile |
| Exchange Manifest | Canonical | Kristal compiler | Verifiers, query systems | manifest content ref + `exchange_hash` | `schemas/exchange-manifest.schema.json` | Deterministic canonicalization; content-addressed identity; binds pipeline refs and policy/blueprint hashes |
| **Kristal Exchange** | Canonical | Kristal compiler | Architect-Render, query engines, exporters | `exchange_id` (aka `kristal_id`) + `exchange_hash` | compiled exchange payload | Canonical truth after validate+compile; immutable; auditable |
| Runtime Pack Manifest | Derived | Kristal compiler / Orgo | Konnaxion | manifest content ref + hashes | `schemas/runtime-pack-manifest.schema.json` (**TBD**) | Verify signatures/hashes fail-closed; atomic activation; deterministic rollback |
| **Runtime Pack** | Derived | Kristal compiler / Orgo | Konnaxion, client runtimes | `pack_id` + pack hashes | pack payload + manifest | Orgo records `pack_id` and payload hashes; activation requires compatibility checks |
| Channel Pack Index (`pack_index.json`) | Derived (recommended) | Distribution channel tooling | Konnaxion | index signature id | JSON | If signed: verify fail-closed; includes `latest`, `pinned`, `revoked` |
| Build Record | Operational | Orgo | Audit, Ops, reproducibility tooling | `build_id` | `schemas/build-record.schema.json` (**TBD**) | Persists inputs, config hashes, policy ids, stage timeline, outputs (exchange/pack ids + hashes/signatures) |
| Release Record (distribution intent) | Operational | Orgo | Distribution / rollout systems | `release_id` | `schemas/release-record.schema.json` (**TBD**) | Versioned object tying build → channels/cohorts; verification/activation status tracked |
| Orgo Case | Operational | Orgo | Orgo services, UIs | `case_id` | `docs/04-artifacts-contracts/08-orgo-case-task.md` (+ optional schema) | Long-lived container grouping context and tasks; status lifecycle enforced |
| Orgo Task | Operational | Orgo | Orgo services, SwarmCraft | `task_id` | `schemas/orgo-task.schema.json` + `docs/04-artifacts-contracts/08-orgo-case-task.md` | Task is canonical unit of work; strict shared schema at API boundaries; status machine + terminal semantics |
| Task/Case Label String | Operational | Orgo | Routing, visibility, analytics | `label` | string | Canonical format: `<BASE>.<CATEGORY>.<SUBCATEGORY>.<HORIZONTAL_ROLE...>` (dot-separated; role is one or more segments) |
| Plan → Cases/Tasks mapping | Operational | Architect-Strategy | Orgo, SwarmCraft | `plan_id` (**TBD**) | plan envelope (**TBD**) | Strategy proposes work; Orgo enforces lifecycle; execution emits telemetry; no direct canon mutation |
| Render Bundle | Derived (normative interface) | Architect-Render | Konnaxion/UI, audit store | `bundle_id` | `schemas/render-bundle.schema.json` + `docs/04-artifacts-contracts/07-render-bundle.md` | Must include outputs + `trace_map`; no-new-facts; deterministic under pinned inputs/template/params |
| Trace Map | Derived (normative interface) | Architect-Render | Audit/verification tooling | assertion ids | embedded in Render Bundle | Every factual statement: TRACED with supports, or OMITTED/UNCERTAIN/REFUSED deterministically |
| Render Events (omissions/refusals/uncertainties) | Derived (normative interface) | Architect-Render | Callers/UIs | codes + ids | embedded in Render Bundle (`events.*`) | Deterministic codes; stable mapping to assertion ids/spans/anchors |
| Activation Record (local) | Operational | Konnaxion | Ops/telemetry | active `pack_id` | local metadata | Activate only after verify+schema+compat; atomic switch; downgrade prevention |
| Rollback Record / Mode | Operational | Konnaxion | Ops | target `pack_id` | local metadata | Supports pinned or last-known-good rollback; deterministic given same triggers |
| Distribution Cache Layout | Operational | Konnaxion | Local runtime | filesystem paths | storage contract | Multi-version storage; atomic activation; deterministic cache policies; offline serving |
| Konnaxion Telemetry Signals | Operational | Konnaxion | Orgo | event ids | structured events | Signals must not mutate Exchange; should create Cases/Tasks or distribution adjustments |
| EkoH: UserExpertiseScore | Operational ledger | EkoH | Profiles, weighting, moderation | row ids | DB table | Current expertise per domain; used for weighted influence |
| EkoH: UserEthicsScore | Operational ledger | EkoH | Weighting engines | user PK | DB table | Ethical multiplier applied to weights |
| EkoH: ScoreHistory | Operational ledger | EkoH | Audit/UX | row ids | DB table | Immutable audit trail (append corrections; don’t rewrite) |
| Smart Vote: Vote / VoteResult | Operational ledger | Smart Vote | EkoH, UIs | row ids | DB tables | Raw votes → weighted outcomes; aggregated VoteResult produced and consumed |
| Privacy: ConfidentialitySetting | Operational | Konnaxion/EkoH | UIs | user PK | DB table | Privacy levels must be honored (public/pseudonym/anonymous or equivalent policy) |

---

## 4) Under-specified (TBD) artifacts to formalize

The following exist conceptually but require a single canonical schema to be “tight”:

- Input Snapshot manifest schema
- Mandate / policy bundle schema (Keter bundle)
- Blueprint bundle schema (schema + policy/config bundle)
- Architect-Strategy Plan Envelope schema (Strategy → Orgo)
- SwarmCraft task execution envelope schema (Orgo → SwarmCraft)
- Build Record schema
- Release Record schema
- Runtime Pack manifest schema (if not yet locked)
- EkoH portable ledger export (only if needed beyond DB)

---

## 5) Minimum conformance expectations (catalog-level)

- Orgo enforces stage order and gate semantics, and records content-addressed stage outputs.
- Validation failure produces no Exchange update (“no compile on fail”).
- Distribution/activation verification is fail-closed; rollback deterministic.
- Rendering introduces no new facts and provides trace coverage, or omits/marks uncertain/refuses deterministically per policy.
