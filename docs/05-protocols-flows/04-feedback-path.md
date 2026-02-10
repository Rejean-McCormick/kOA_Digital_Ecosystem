# Feedback Path (Protocol) — Signals → Governed Work → New Canon (Never Direct Mutation)

This document specifies the **Feedback Path**: how user/community/system signals re-enter the ecosystem as **governed work** without mutating canon in place. Feedback becomes actionable only through Orgo Cases/Tasks and, if it affects truth, through the full pipeline (Extract → Resolve → Validate → Compile).

Canon is Kristal Exchange. Feedback never edits it directly.

---

## 1) Purpose

The Feedback Path exists to:
- capture corrections, disputes, improvements, and operational signals
- convert signals into **governed** work items (Cases/Tasks)
- preserve accountability and traceability (who said what, why, and when)
- ensure truth changes occur only through deterministic gates (validation + compile)

---

## 2) Actors (nodes) and responsibilities

- **Konnaxion (Chesed)**: collects feedback in social surfaces; enforces identity/privacy; routes signals.
- **EkoH (Malkuth)**: aggregates trust/impact signals; computes weights; provides audit trails.
- **Orgo (Gevurah)**: triages, classifies, and turns feedback into Cases/Tasks; enforces policy and gates.
- **SenTient (Tiferet)**: resolves conflicting claims and preserves ambiguity when needed.
- **Kristal (Da’at)**: compiles new canon only after validation PASS.
- **Architect (Netzach/Hod)**: may render summaries of feedback and outcomes; may propose plans (Strategy).

---

## 3) Core invariants (must always hold)

1) **No direct canon mutation**: feedback cannot edit Kristal Exchange in place.
2) **Governance first**: feedback becomes action only as Orgo Cases/Tasks.
3) **Traceability**: all feedback is attributable (or explicitly anonymous) and auditable.
4) **Ambiguity discipline**: unresolved disputes remain explicit; do not force resolution.
5) **Deterministic gates**: truth-impacting changes require validation PASS and compilation.
6) **Separation of concerns**: trust signals influence prioritization, not truth content directly.

---

## 4) Types of feedback (signals)

Feedback is classified into four types with different handling paths:

### 4.1 Truth-impacting
- correction of a claim
- new evidence contradicting canon
- dispute about identity/relations/time constraints
- request to mark a fact as uncertain/contested

### 4.2 Scope/policy-impacting
- mandate change requests
- blueprint/schema/policy change requests
- access/privacy/confidentiality corrections

### 4.3 Product/UX-impacting (non-truth)
- rendering issues (trace gaps, refusal UX)
- formatting/translation problems
- usability and navigation issues

### 4.4 Operational signals (non-truth)
- distribution failures, verification failures, rollback events
- execution errors, performance regressions
- telemetry anomalies

---

## 5) Feedback artifacts (contracts)

### 5.1 Feedback Event (minimum)
A normalized envelope that can be stored, routed, and audited.

Fields (minimum):
- `feedback_id`
- `tenant_id` / `channel_id` (if applicable)
- `actor` (user id or `anonymous`, plus confidentiality mode)
- `feedback_type` (truth | policy | ux | ops)
- `target_refs[]` (optional: exchange_id, pack_id, render_bundle_id, claim ids, task ids)
- `message` (free text) + optional structured fields
- `attachments[]` (links or uploaded evidence references)
- `created_at`

### 5.2 Correction Proposal (truth-impacting)
A structured proposal that points to canon and asserts what should change.

Fields (minimum):
- `proposal_id`
- `target_exchange_id`
- `target_claim_ref` (or slot ref)
- `proposed_change` (add | remove | modify | mark_uncertain | mark_contested)
- `evidence_refs[]` (input snapshots and pointers)
- `rationale` (structured short)
- `confidence` (optional)
- `status` (submitted | triaged | accepted_for_work | rejected | merged)

### 5.3 Triage Decision (Orgo)
A record binding a feedback item to the work it creates.

Fields (minimum):
- `triage_id`
- `feedback_id`
- `decision` (ignore | request_info | create_task | create_case | escalate_security | open_policy_change)
- `reason_codes[]`
- `created_case_id` / `created_task_ids[]` (if any)

---

## 6) Protocol sequence (canonical)

### 6.1 Capture and normalize (Konnaxion)
1) User/system submits feedback (UI/API/telemetry).
2) Konnaxion normalizes into a Feedback Event.
3) Konnaxion applies:
   - identity and confidentiality rules
   - spam/abuse filters (policy)
4) Konnaxion forwards to Orgo for triage (or queues for moderation if required).

### 6.2 Triage and classification (Orgo)
Orgo MUST:
1) classify feedback type (truth | policy | ux | ops)
2) validate minimal required fields
3) attach target references (if missing, request info)
4) decide the action:
   - request more info
   - create a Case/Task
   - escalate (security/moderation)
   - mark as duplicate and link to existing Case

Truth-impacting feedback MUST create governed work that re-enters the truth pipeline.

### 6.3 Prioritization (EkoH + Orgo)
If enabled:
- EkoH provides trust/impact weighting signals (expertise/ethics, vote outcomes).
- Orgo uses weights to prioritize and route work.
- Weighting MUST NOT directly change canon; it only affects ordering and attention.

### 6.4 Work execution (Execution Path)
Created tasks follow the Execution Path:
- Architect-Strategy may propose a plan
- Orgo gates and dispatches
- SwarmCraft executes and emits telemetry

### 6.5 Truth change handling (full pipeline)
If the Case is truth-impacting:
1) ingest new evidence as input snapshots
2) extract Claim-IR
3) resolve into Resolved Claim-IR (SenTient)
4) validate deterministically (Orgo gate)
5) compile new Exchange + Runtime Pack (Kristal)
6) distribute new pack (Konnaxion), render updated outputs (Architect-Render)

No step may be skipped.

### 6.6 Closing the loop
- Orgo marks the feedback item as resolved and links:
  - the resulting Case/Tasks
  - any new exchange_id/pack_id
  - render bundle(s) produced
- Konnaxion surfaces the outcome to the user (respecting confidentiality).

---

## 7) Gate rules (normative)

### 7.1 “No in-place edit” gate
- Feedback MUST NOT trigger direct edits of Kristal Exchange.
- Any truth change MUST produce a new exchange_id (new canonical artifact).

### 7.2 “Evidence required” gate (truth-impacting)
- Truth-impacting correction proposals MUST include evidence references or be converted into “request evidence” tasks.
- Orgo MUST refuse to treat unsupported corrections as truth changes.

### 7.3 “Ambiguity preserved” gate
- If evidence is insufficient to resolve a dispute, the system MUST:
  - preserve ambiguity (contested/unknown)
  - or refuse to change canon
- It MUST NOT silently select a side.

### 7.4 “Policy change requires ADR” gate (optional but recommended)
- Any change to invariants, schemas, or gate rules MUST be recorded as an ADR before adoption.

---

## 8) Failure modes (expected behavior)

- Missing target reference: request info; do not guess targets.
- Spam/abuse: quarantine or moderation queue; no Orgo execution.
- Conflicting high-confidence claims: create dispute Case; preserve ambiguity until resolved.
- Low-evidence correction: record, request evidence, or reject with reason codes.
- System telemetry anomaly: create ops Case; do not treat as truth change.

---

## 9) Observability requirements

Minimum telemetry for feedback handling:
- `feedback_id` correlation through triage, tasks, and outcomes
- triage decision codes and created Case/Task IDs
- time-to-triage, time-to-resolution
- counts by feedback type and outcome category
- duplication rates, spam quarantine rates
- truth-impacting conversion rate (feedback → new exchange_id)

---

## 10) Conformance tests (required)

A conformant implementation MUST test:
- feedback never edits canon in place (always produces new work artifacts)
- truth-impacting feedback creates Cases/Tasks and re-enters full pipeline
- evidence requirement enforcement (request_info or reject without evidence)
- ambiguity preservation (no forced resolution)
- confidentiality enforcement in surfaces and logs
- deterministic triage rules (if configured) and stable reason codes
