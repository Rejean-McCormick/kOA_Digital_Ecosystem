# Lifecycle (End-to-End)

This document defines the canonical lifecycle of the ecosystem: how raw inputs become validated canonical truth (Kristal Exchange), how that truth becomes portable execution artifacts (Runtime Packs), how outputs are rendered deterministically, how work is executed, and how feedback re-enters the system without mutating canon.

---

## 1. Key terms

- **Build**: a governed run of the pipeline producing a new Kristal Exchange + Runtime Pack (or failing before canon).
- **Release**: a distributed publication of a Runtime Pack (and associated metadata) through Konnaxion.
- **Case / Task**: Orgo’s governance units. A **Task** is the atomic unit of work; a **Case** groups tasks over time.
- **Canon**: Kristal Exchange (the authoritative, immutable truth artifact).
- **Derived**: Runtime Pack and Render Bundles (must be traceable to canon + pinned configuration).
- **Gate**: a deterministic acceptance point; failure blocks downstream stages (fail-closed when integrity is declared).

---

## 2. Lifecycle overview

### High-level flow
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

### Mermaid flow (conceptual)
```mermaid
flowchart TD
  A[Mandate + Blueprint] --> B[Ingest Inputs]
  B --> C[Extract -> Claim-IR]
  C --> D[Resolve -> Resolved Claim-IR]
  D --> E{Validate?}
  E -- fail --> X[Stop: No Canon / No Pack / No Release]
  E -- pass --> F[Compile -> Kristal Exchange + Runtime Pack]
  F --> G{Verify for Activation?}
  G -- fail --> Y[Reject: Fail-Closed / Rollback-or-Stay]
  G -- pass --> H[Distribute Runtime Pack]
  H --> I[Render -> Render Bundle + trace_map]
  I --> J[Execute Tasks -> Telemetry]
  J --> K[Feedback -> New Case/Task]
  K --> C
````

---

## 3. Stage-by-stage specification

Each stage is defined by: Inputs, Outputs, Owner, Gate conditions, and Required logs.

### Stage 0 — Mandate + Blueprint load

**Owner**: Mandate node + Blueprint node

**Inputs**

* Mandate bundle (purpose, scope, constraints, success criteria)
* Blueprint bundle (schemas, ontologies, policies, versions)

**Outputs**

* `runtime_context_ref` (immutable reference bundle to mandate + blueprint pins)

**Gate**

* MUST refuse to run the pipeline without an active mandate (when policy requires it) and pinned blueprint versions.
* MUST record the exact mandate/policy and blueprint hash used for the build.

---

### Stage 1 — Ingest (raw inputs)

**Owner**: Inputs node (Ingest adapters under Orgo control)

**Inputs**

* source documents, feeds, user uploads, connectors

**Outputs**

* `input_snapshots[]` (immutable snapshot refs)
* `input_snapshot_set_ref` (recommended manifest listing snapshots + provenance)
* optional `ingest_report_ref`

**Gate**

* provenance MUST be preserved
* snapshots MUST be immutable (content-addressed preferred)

**Logs (minimum)**

* ingest adapter version/config hash
* snapshot IDs + snapshot_set_ref
* provenance pointers (opaque refs when sensitive)

---

### Stage 2 — Extract → Claim-IR

**Owner**: Extractor(s) (Chokmah facet; governed by Orgo)

**Inputs**

* input snapshots (or snapshot set ref)
* Claim-IR schema + extraction policy (pinned blueprint/policy)

**Outputs**

* `claim_ir_ref` (content-addressed batch ref)
* optional `extract_report_ref`

**Gate**

* MUST NOT output canonical “truth”
* MUST NOT collapse ambiguity present in sources
* MUST include evidence pointers and uncertainty markers where applicable

**Logs (minimum)**

* extractor version, config hash, policy pins
* input snapshot refs
* claim_ir_ref

---

### Stage 3 — Resolve → Resolved Claim-IR (SenTient)

**Owner**: SenTient

**Inputs**

* `claim_ir_ref`
* blueprint resolution rules + policy selections (pinned)

**Outputs**

* `resolved_claim_ir_ref`
* optional `resolution_report_ref`

**Gate**

* MUST keep ambiguity explicit when resolution is not justified
* MUST NOT introduce new facts (everything must trace to upstream claim evidence)
* MUST express explicit decision states (resolved/unresolved/rejected/merged)

**Logs (minimum)**

* SenTient version/config hash + determinism mode
* policy pins + claim_ir_ref → resolved_claim_ir_ref

---

### Stage 4 — Validate (deterministic gate)

**Owner**: Orgo (control plane gate)

**Inputs**

* `resolved_claim_ir_ref`
* validation policy + blueprint constraints (pinned)

**Outputs**

* `validation_report_ref`
* derived `validation_status` (passed|partial|failed)

**Gate**

* **NO COMPILE ON FAIL**
* validation MUST be deterministic given identical inputs + policy versions
* fail-closed when required checks cannot be performed deterministically

**Logs (minimum)**

* policy_id, blueprint_hash, validator identity/config hash
* validation_status + reason codes
* validation_report_ref

---

### Stage 5 — Compile (Kristal canon + runtime)

**Owner**: Kristal (canon pivot)

**Inputs**

* `validation_report_ref` (MUST be PASS, unless policy explicitly allows PARTIAL)
* `resolved_claim_ir_ref`
* compiler config + canonicalization profile (pinned)

**Outputs**

* **Kristal Exchange**: `exchange_id`/`kristal_id` + `exchange_manifest_ref`
* **Runtime Pack**: `pack_id` + `runtime_pack_manifest_ref`
* integrity material: hashes/signatures (as configured)

**Gate**

* MUST be reproducible under deterministic mode (same inputs → same IDs/hashes)
* MUST record manifests binding inputs/policy/config → outputs

**Logs (minimum)**

* compiler version/config hash
* exchange_id, pack_id, manifest refs, integrity results

---

### Stage 6 — Publish / Distribute (Konnaxion)

**Owner**: Konnaxion (distribution facet)

**Inputs**

* Runtime Pack + manifest refs
* distribution policy (channels, visibility, activation rules)
* tenant trust roots / verification policy

**Outputs**

* `release_id` (release record)
* distribution timeline (queued → delivered → verified → activated)
* failure codes (if any)

**Gate**

* pack verification MUST happen before activation
* verification failure MUST be fail-closed (no partial activation)
* rollback MUST be supported to last-known-good (or remain pinned) where applicable

**Logs (minimum)**

* release_id + channel/cohort targets
* verify pass/fail + cause codes
* activation/rollback events + resulting active pack_id

---

### Stage 7 — Render (Architect-Render)

**Owner**: Architect-Render

**Inputs**

* Kristal Exchange and/or Runtime Pack (pinned)
* template id/version (pinned), locale, rendering parameters
* strictness profile (pinned)

**Outputs**

* **Render Bundle**: output payload(s) + `trace_map` + deterministic context metadata

**Gate**

* rendering MUST be deterministic given same inputs/template/params/engine config
* MUST NOT introduce new facts
* every factual statement MUST be traceable; otherwise:

  * omit, or
  * mark uncertain (only if uncertainty exists upstream), or
  * refuse deterministically (strict profile)

**Logs (minimum)**

* render_bundle_id, template pins, engine config hash
* trace coverage metrics + refusal codes (if any)

---

### Stage 8 — Execute (SwarmCraft)

**Owner**: SwarmCraft

**Inputs**

* Orgo Tasks (typed work)
* runtime context + pack references (pinned)

**Outputs**

* execution results (typed artifacts)
* telemetry/logs (correlation IDs, step status, produced artifacts)

**Gate**

* execution MUST preserve auditability (who/what/when/with-what-config)
* artifacts produced MUST be recorded with ids + hashes (when applicable)

---

### Stage 9 — Feedback → new governed work

**Owner**: Orgo + Konnaxion (social facet) + EkoH

**Inputs**

* user feedback, votes, corrections, trust signals

**Outputs**

* new Cases/Tasks (governed work items)
* ledger updates (trust/reputation), never direct canon mutation

**Gate**

* feedback MUST NOT mutate Kristal Exchange directly
* changes MUST re-enter through governed pipeline stages (2–5)

**Logs (minimum)**

* feedback_id → case_id/task_id linkage
* any ledger append references (no raw private payloads)

---

## 4. Global gates (hard stops)

1. **Validation gate**: Stage 4 fail ⇒ Stage 5 MUST NOT run (no Exchange, no Pack).
2. **Verification gate**: Stage 6 fail ⇒ no activation; rollback or remain on last-known-good/pinned.
3. **Rendering gate**: Stage 7 must be deterministic and trace-complete under policy; otherwise omit/uncertain/refuse deterministically.

---

## 5. Determinism and identity

* Artifact IDs SHOULD be content-derived where feasible (content-addressed).
* Every build MUST produce a manifest binding:

  * mandate version + blueprint hash
  * input snapshot set ref / snapshot IDs
  * policy_id + policy selections refs
  * component identities (versions + config hashes)
  * resulting `exchange_id` and `pack_id`

This ensures replayability and auditability.

---

## 6. Observability requirements (minimum)

Every stage MUST emit telemetry including:

* `tenant_id`, `environment`
* `build_id`, `stage_id`, `attempt_id`, `correlation_id`
* input artifact refs and output artifact refs
* component version + config hash
* deterministic decision traces for gates (validation, compile refusal, render refusal)

Recommended metrics:

* stage durations, gate pass/fail rates
* verification failures by cause
* render refusals due to trace gaps
* rollback events

---

## 7. Failure modes (expected behavior)

* **Ambiguous input**: preserve ambiguity; do not force resolution.
* **Validation fail**: stop; produce validation artifacts only; no Exchange/Pack/Release.
* **Pack verification fail**: fail-closed; do not activate; rollback or remain pinned.
* **Trace gap during render**: omit/mark uncertain if upstream uncertainty exists; otherwise refuse deterministically.
* **Feedback contradiction**: open a governed Case/Task; never mutate canon in place.

---

## 8. Minimal appendix: where each concern lives

* Governance and gates: **Orgo**
* Canon and compilation: **Kristal**
* Deterministic articulation: **Architect-Render**
* Distribution safety: **Konnaxion**
* Execution: **SwarmCraft**
* Trust/impact ledger: **EkoH**
* Meaning resolution: **SenTient**
