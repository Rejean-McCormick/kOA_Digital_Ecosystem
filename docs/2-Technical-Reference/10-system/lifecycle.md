# Lifecycle (kOA end-to-end)

**Status:** Canonical (system-level)
**Purpose:** Define the canonical lifecycle of the ecosystem: how raw inputs become validated canonical truth (Kristal Exchange), how that truth becomes portable offline execution artifacts (Runtime Packs), how outputs are rendered deterministically, how work is executed, and how feedback re-enters the system without mutating canon.

---

## 1) Key terms

- **Build:** A governed run of the pipeline producing a new Kristal Exchange + Runtime Pack (or failing before canon).
- **Release:** A distributed publication of a Runtime Pack (and associated metadata) through Konnaxion.
- **Case / Task:** Orgo’s governance units. A **Task** is the atomic unit of work; a **Case** groups tasks over time.
- **Canon:** Kristal Exchange (the authoritative, immutable truth artifact).
- **Derived:** Runtime Pack and Render Bundles (must be traceable to canon + pinned configuration).
- **Gate:** A deterministic acceptance point; failure blocks downstream stages (fail-closed when integrity is declared).

---

## 2) Lifecycle overview

### 2.1 High-level stage spine

0) Load Mandate + Blueprint  
1) Ingest inputs  
2) Extract Claim-IR  
3) Resolve Claim-IR (SenTient)  
4) Validate (Orgo gate)  
5) Compile canon + runtime (Kristal)  
6) Publish/Distribute (Konnaxion)  
7) Render outputs (Architect-Render)  
8) Execute work (SwarmCraft)  
9) Feedback → new governed work (Orgo)

### 2.2 Mermaid (conceptual)

```mermaid
flowchart TD
  A[Mandate + Blueprint] --> B[Ingest Inputs]
  B --> C[Extract -> Claim-IR]
  C --> D[Resolve -> Resolved Claim-IR]
  D --> E{Validate?}
  E -- fail --> X[Stop: No Canon / No Pack / No Release]
  E -- pass --> F[Compile -> Exchange + Runtime Pack]
  F --> G{Verify for Activation?}
  G -- fail --> Y[Reject: Fail-Closed / Rollback-or-Stay]
  G -- pass --> H[Distribute Runtime Pack]
  H --> I[Render -> Render Bundle + trace_map]
  I --> J[Execute Tasks -> Telemetry]
  J --> K[Feedback -> New Case/Task]
  K --> C
````

---

## 3) Stage-by-stage specification

Each stage specifies: **Owner**, **Inputs**, **Outputs**, **Gate**, **Observability**.

### Stage 0 — Mandate + Blueprint load

**Owner:** Keter (Mandate) + Binah (Blueprint), enforced by Orgo.

**Inputs**

* Mandate bundle (scope, constraints, success criteria)
* Blueprint bundle (schemas, ontologies, policies, versions)

**Outputs**

* Pinned runtime context reference (immutable refs to mandate + blueprint)

**Gate**

* Must refuse to run without an active mandate (when required by policy).
* Must pin exact blueprint revisions used for the build.

**Observability**

* mandate_ref, blueprint_ref, policy pins
* correlation IDs (build_id / request_id if present)

---

### Stage 1 — Ingest (raw inputs)

**Owner:** Chokmah (Inputs adapters), orchestrated by Orgo.

**Inputs**

* Source documents, feeds, uploads, connectors
* Ingest policy + provenance rules (from Blueprint)

**Outputs**

* Input snapshots (immutable snapshot refs)
* Optional snapshot set manifest (listing snapshots + provenance pointers)

**Gate**

* Provenance must be preserved.
* Snapshots must be immutable (content-addressed strongly recommended).

**Observability**

* ingest adapter version/config ref
* snapshot IDs, snapshot-set ref
* provenance pointers (opaque refs allowed)

---

### Stage 2 — Extract → Claim-IR

**Owner:** Extractor(s), governed by Orgo.

**Inputs**

* Input snapshots (or snapshot-set ref)
* Claim-IR schema/policy (pinned from Blueprint)

**Outputs**

* Claim-IR batch reference

**Gate**

* Extractors must output proposals only (no “truth”).
* Evidence pointers and uncertainty markers must be preserved.

**Observability**

* extractor identity/version/config ref
* input snapshot refs used
* claim-ir ref

---

### Stage 3 — Resolve → Resolved Claim-IR (SenTient)

**Owner:** SenTient, governed by Orgo.

**Inputs**

* Claim-IR batch
* Resolution policy + candidate sources (pinned)

**Outputs**

* Resolved Claim-IR batch reference (with explicit ambiguity preserved)

**Gate**

* Must be deterministic under pinned policy/config.
* Must preserve unresolved ambiguity explicitly (no silent coercion).

**Observability**

* resolver identity/version/config ref
* warnings/errors summary (stable codes)
* resolved-claim-ir ref

---

### Stage 4 — Validate (deterministic acceptance gate)

**Owner:** Validation engine, enforced by Orgo.

**Inputs**

* Resolved Claim-IR
* Validation policy/profile set (pinned)

**Outputs**

* Validation Report reference

**Gate (hard)**

* If validation fails, compilation must not proceed (“no compile on fail”).

**Observability**

* validator identity/version/config ref
* validation report ref
* gate decision + reason codes

---

### Stage 5 — Compile (Kristal)

**Owner:** Kristal compiler, invoked by Orgo after Stage 4 pass.

**Inputs**

* Resolved Claim-IR
* Validation Report (pass)
* Compiler config + pinned profiles/policies

**Outputs**

* Kristal Exchange (+ Exchange Manifest)
* Runtime Pack (+ Runtime Pack Manifest)

**Gate**

* Must produce schema-conformant artifacts (Kristal-owned schemas).
* Must record reproducibility metadata sufficient for rebuild determinism (via manifests/build records).

**Observability**

* compiler identity/version/config ref
* exchange refs + pack refs
* manifest refs
* content-hash / signature verification results (if produced)

---

### Stage 6 — Publish / Distribute (Konnaxion)

**Owner:** Konnaxion (distribution facet), triggered and tracked by Orgo.

**Inputs**

* Runtime Pack + manifest
* Channel/release intent (from Orgo)
* Trust roots / revocation info (tenant-scoped)

**Outputs**

* Published artifacts in channel(s)
* Activation/verification status updates
* Rollback records (when applicable)

**Gate (hard)**

* If integrity material is declared (hash/signatures), verification must be fail-closed.
* Activation must be atomic; rollback must be deterministic.

**Observability**

* verification results (pass/fail + codes)
* active pack ID, previous pack ID
* rollout cohort/channel + timestamps

---

### Stage 7 — Render outputs (Architect-Render)

**Owner:** Architect-Render.

**Inputs**

* Kristal query results (from Exchange and/or Pack, per policy)
* Render template + parameters (pinned)
* Render policy (no-new-facts + trace requirements)

**Outputs**

* Render Bundle (includes trace coverage)

**Gate (hard)**

* Rendering must not introduce new facts.
* Output must be deterministic under pinned inputs/template/params.
* If trace coverage is insufficient, renderer must deterministically omit / mark-uncertain / refuse per policy.

**Observability**

* renderer identity/version/config ref
* template + params refs
* render bundle ref + trace coverage metrics

---

### Stage 8 — Execute work (SwarmCraft)

**Owner:** SwarmCraft (or equivalent execution plane), governed by Orgo.

**Inputs**

* Orgo Tasks (and required inputs)
* Execution policy (resource caps, tool allow-lists)

**Outputs**

* Execution results
* Telemetry events

**Gate**

* Must follow deterministic policy where required (stable toolchain versions, fixed seeds when applicable).
* Must emit telemetry sufficient for audit and feedback routing.

**Observability**

* task_id, case_id, correlation IDs
* result refs
* telemetry refs

---

### Stage 9 — Feedback → new governed work

**Owner:** Orgo (ingestion of signals).

**Inputs**

* Telemetry, user feedback, ops events, trust/impact signals

**Outputs**

* New Cases/Tasks (governed work items)
* Optional prioritization/triage updates

**Gate**

* Feedback must not mutate canon directly.
* Any canon-affecting change must be a new build (new artifacts, new IDs).

**Observability**

* signal source + classification
* created case/task IDs + routing
* linkage to triggering build/release IDs (when applicable)

---

## 4) Cross-cutting hard rules (summary)

* **No compile on fail:** Stage 4 fail blocks Stage 5.
* **Fail-closed integrity:** Any declared hashes/signatures must verify before activation/publish.
* **Immutable canon:** Exchange is never edited in place; updates produce new artifacts/IDs.
* **No new facts downstream:** Rendering cannot invent; it must trace or refuse deterministically.

---

## 5) Pointers

* Build orchestration details: `docs/50-operations/pipeline.md`
* Kristal v4 integration profile and contract pointers: `docs/40-integration/kristal-v4/`
* Conformance tests: `docs/60-guides/testing.md`


