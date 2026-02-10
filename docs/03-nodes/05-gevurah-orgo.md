# Gevurah — Orgo (Control Plane)

**File:** `docs/03-nodes/05-gevurah-orgo.md`  
**Role:** Workflow orchestration, governance, gating, auditability (ecosystem control plane). :contentReference[oaicite:0]{index=0}

---

## 1) Purpose

Orgo is the system authority that:
- labels and routes information/work across vertical + horizontal axes, :contentReference[oaicite:1]{index=1}
- defines canonical Case/Task contracts at API/message boundaries, :contentReference[oaicite:2]{index=2}
- enforces status lifecycles and allowed transitions, :contentReference[oaicite:3]{index=3}
- runs cyclic/pattern overview that converts threshold crossings into new Cases (governed review work). :contentReference[oaicite:4]{index=4} :contentReference[oaicite:5]{index=5}

For Kristal production, Orgo is the **sole authority** that advances pipeline stages and enforces gating. :contentReference[oaicite:6]{index=6}

---

## 2) Responsibilities (normative)

### 2.1 Workflow control plane for Kristal builds
Orgo MUST: create/manage Kristal build workflows, store workflow state/approvals/audit logs, record build inputs/outputs with content-addressed identifiers, trigger distribution and track distribution status, and surface deterministic errors/warnings to operators. :contentReference[oaicite:7]{index=7}

Orgo MUST NOT: mutate Kristal Exchange artifacts directly as a result of feedback signals; bypass deterministic validation gates. :contentReference[oaicite:8]{index=8} :contentReference[oaicite:9]{index=9}

### 2.2 Unified Case/Task work model (ecosystem governance object)
- **Task is the canonical unit of work.** :contentReference[oaicite:10]{index=10}  
- **Case** is a long-lived container aggregating tasks, patterns, context and review history. :contentReference[oaicite:11]{index=11}

Domain modules MUST NOT create parallel task tables or lifecycles; they are adapters over the global Task/Case engine. :contentReference[oaicite:12]{index=12}

### 2.3 Multi-tenant backbone enforcement
Orgo MUST enforce tenant isolation and org-scoped data boundaries through `organization_id` conventions and access control. :contentReference[oaicite:13]{index=13}

---

## 3) What Orgo is NOT

- Not a system that edits canonical truth outside formal validation+compile. :contentReference[oaicite:14]{index=14}  
- Not a generic CRM/ERP; not a “kanban toy.” :contentReference[oaicite:15]{index=15}  
- Not a replacement for SenTient resolution, Kristal compilation, Architect rendering, or Konnaxion distribution (it orchestrates them). :contentReference[oaicite:16]{index=16}

---

## 4) Interfaces (boundaries)

### 4.1 Pipeline systems Orgo orchestrates
Orgo orchestrates interactions with:
- Extractors → Claim-IR
- SenTient → Resolved Claim-IR
- Validation engine (deterministic)
- Kristal compiler → Exchange + Runtime Pack
- Distribution systems (e.g., Konnaxion) :contentReference[oaicite:17]{index=17}

### 4.2 Artifact boundaries (what crosses)
Orgo MUST record content-addressed references to stage outputs including Claim-IR batch IDs, Resolved Claim-IR batch IDs, Validation report IDs, Exchange `kristal_id`, Runtime Pack `pack_id` and payload hashes. :contentReference[oaicite:18]{index=18}

### 4.3 Feedback boundary (governed change)
Feedback/corrections MUST be represented as **new Cases/Tasks** and must not directly edit Exchange. :contentReference[oaicite:19]{index=19}

---

## 5) Orgo data model essentials

### 5.1 Tenancy key
- Tenancy key: `organization_id` (org-scoped records have non-NULL `organization_id`; global defaults may use NULL and be overridden per org). :contentReference[oaicite:20]{index=20}

### 5.2 Label system (routing + visibility + analytics)
Every Case/Task carries a structured label:

```text
<BASE>.<CATEGORY><SUBCATEGORY>.<HORIZONTAL_ROLE?>
````

Example: `100.94.Operations.Safety` 

Label informs routing, default visibility, and analytics/pattern grouping. 

Broadcast bases `10`, `100`, `1000` are informational by default and do not automatically spawn mandatory tasks unless a workflow rule says so. 

### 5.3 Canonical Case/Task semantics (summary)

* Case: long-lived container for incident/theme; used in cyclic overview reviews. 
* Task: central work unit with strict schema; includes normalized ownership (`owner_role_id`/`owner_user_id`) plus denormalized fields, deadlines (`due_at`, reactivity deadlines), severity/visibility, escalation level, metadata. 

---

## 6) Workflow contract for Kristal builds

### 6.1 Mandatory stage order

A Kristal build workflow MUST follow this stage order: 

1. Ingest
2. Extract → Claim-IR
3. Resolve (SenTient) → Resolved Claim-IR
4. Validate
5. Compile → Exchange
6. Compile → Runtime Pack
7. Publish/Distribute
8. Post-publish verification (recommended)

Orgo MAY split stages into sub-stages but MUST preserve ordering and gating semantics. 

### 6.2 “No compile on fail” (hard gate)

If validation fails:

* Orgo MUST mark the workflow as failed at Validation,
* MUST NOT invoke Exchange or Runtime Pack compilation,
* MUST surface the structured validation report to operators. 

### 6.3 Idempotency and replay

Orgo SHOULD support deterministic replay given the same inputs/config (subject to probabilistic extractors being frozen or replaced with stored Claim-IR) and MUST ensure replay does not bypass integrity checks. 

---

## 7) Build records and release records

### 7.1 Build Record (mandatory)

For every Kristal build, Orgo MUST persist a Build Record containing:

* identifiers: `build_id`, `tenant_id`, `workflow_id` / `case_id`, 
* inputs (snapshots): content-addressed references to all inputs used (datasets, Claim-IR snapshots, config snapshots), 
* compiler identity/config hash/canonicalization profile, 
* policy selections, 
* outputs: `kristal_id`, hashes/signatures; `pack_id`, payload hashes, manifest hash/signature refs. 

### 7.2 Release Record (distribution traceability)

When distribution is triggered, Orgo creates a versioned release object tying a build to deployment parameters (channels/cohorts), enabling traceability build → release → distribution.  

---

## 8) State machines (Cases and Tasks)

### 8.1 Case status lifecycle

`CASE_STATUS`: `open`, `in_progress`, `resolved`, `archived`. 

Allowed transitions include:

* `open` → `in_progress|resolved|archived`
* `in_progress` → `resolved|archived`
* `resolved` → `archived|in_progress` (re-open)
* `archived` terminal; reopening is exceptional and must be audited. 

### 8.2 Task status lifecycle

Canonical `TASK_STATUS`: `PENDING`, `IN_PROGRESS`, `ON_HOLD`, `COMPLETED`, `FAILED`, `ESCALATED`, `CANCELLED`. 

Allowed transitions are locked to the core state machine (examples include `PENDING→IN_PROGRESS`, `IN_PROGRESS→COMPLETED|FAILED|ESCALATED`, etc.). 

Normative: when entering a terminal state (`COMPLETED`, `FAILED`, `CANCELLED`), `closed_at` MUST be set; otherwise `closed_at` MUST remain null. 

---

## 9) Cyclic overview (patterns become work)

Orgo’s cyclic overview system continuously computes patterns and threshold crossings and turns them into new Cases for review rather than only analytics. 

---

## 10) Security and integrity responsibilities (offline-safe)

### 10.1 Key/signature metadata and enforcement

Orgo (control plane):

* MUST record build_id, artifact IDs, signer `key_id`, signature metadata,
* SHOULD enforce “no publish without signature” where signatures are required,
* SHOULD manage rotation schedules and revocation list publication. 

### 10.2 Tenant-safe trust roots

Multi-tenant isolation and correct per-tenant trust roots are required to prevent cross-tenant leakage and signature confusion.  

---

## 11) Reliability patterns (recommended operational guidance)

These are operational guidance patterns intended to preserve system stability; they must not change artifacts (only scheduling/control). 

### 11.1 Circuit breaker for SenTient resolution (per tenant)

Use per-tenant circuit breakers for resolution calls; if policy allows, preserve unresolved ambiguity rather than blocking the pipeline indefinitely. 

### 11.2 Dead letter queues (DLQ)

Use staged queues with bounded retries and DLQ; DLQ items should create Orgo Cases/Tasks for triage. 

### 11.3 Timeouts, cancellation, quotas, backpressure

Stages require bounded execution time; validation timeout is failure (“no compile”), and cancellation must not produce partially published artifacts. 
Enforce per-tenant quotas at Orgo admission control and use explicit backpressure signals.  

### 11.4 Release safety

Use canary/blue-green where relevant and always support rollback to last-known-good release subject to downgrade policy. 

---

## 12) Observability (required identifiers)

Recommended required operational identifiers (guidance): `build_id`, `kristal_id`/`exchange_id`, `runtime_pack_id`, `claim_id`, `input_ref`, `tenant_id`. 

Log events at: stage start/end, circuit breaker transitions, retries/DLQ moves, validation failure summaries (bounded). 

---

## 13) Failure modes (must be deterministic at gates)

Common failure classes:

* Invalid Case/Task state transition → reject + log validation error. 
* Validation failure → hard stop; surface structured report; no compile. 
* Integrity verification mismatch (hash/signature) → fail closed (do not publish). 
* Resolution timeout/unavailable → circuit breaker; preserve unresolved if policy allows or stop with reason. 
* Poison inputs → DLQ + triage work items (Case/Task). 

---

## 14) Conformance checklist (Orgo)

An Orgo implementation is conformant to this ecosystem if it:

* enforces stage order and gating (including “no compile on fail”),  
* persists Build Records with required fields, 
* never mutates Exchange from feedback and never bypasses validation gates, 
* enforces tenant isolation with correct tenancy keys and access boundaries, 
* enforces Case/Task state machines and logs invalid transitions, 
* emits the recommended correlation identifiers for auditability. 


