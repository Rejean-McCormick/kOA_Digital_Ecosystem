# Terminology (kOA)

**File:** `docs/90-reference/terminology.md`  
**Purpose:** Provide a concise, kOA-first vocabulary with explicit mapping to external Kristal terms where applicable.  
**Rule:** If a term is defined normatively in Kristal v4, kOA references it rather than redefining it.

---

## 1) kOA ecosystem terms (system-level)

### Artifact
A typed payload exchanged at a boundary. In kOA, artifacts are the only way truth, governance state, and operational intent move between components.

### Truth boundary
The point where “truth” becomes canonical: after deterministic validation and compilation into a Kristal Exchange.

### Gate
A deterministic stage check enforced by Orgo (and/or other control-plane nodes) that can stop the pipeline.

### No compile on fail
Gate rule: if validation fails, compilation/publishing of canonical truth artifacts must not occur.

### Control plane
The governance/orchestration layer (primarily Orgo) that enforces stage order, gates, auditability, and release intent.

### Data plane
The content and distribution layer (artifacts and payloads) that is produced/verified/served to consumers.

---

## 2) Nodes (kOA component names)

### Orgo
kOA governance and orchestration node. Enforces stage order, gating, audit records, and release workflows.

### Konnaxion
Connectivity and distribution node. In distribution mode: verifies, activates, rolls back Runtime Packs fail-closed.

### SenTient
Resolution engine that converts Claim-IR → Resolved Claim-IR while preserving uncertainty.

### Architect-Strategy
Planning node that proposes work (cases/tasks) and objectives; does not publish truth.

### Architect-Render
Deterministic renderer that produces user-facing outputs with traceability and no-new-facts constraints.

### SwarmCraft
Execution node that runs governed tasks and emits telemetry; does not mutate canonical truth directly.

### EkoH
Ledger and scoring subsystem (trust/ethics/weights) used for governance and influence.

### Kristal (truth pivot)
Compilation boundary that produces canonical Exchange and derived Runtime Packs.

---

## 3) kOA-native artifacts (owned by kOA)

### Orgo Case
Operational container grouping context and tasks under a lifecycle.

### Orgo Task
Canonical unit of work for execution/processing; has strict lifecycle semantics at APIs.

### Build Record
kOA record of a pipeline run: pinned inputs, pinned policies/resources, stage outputs, and references to published artifacts.

### Release Record
kOA record of distribution intent and rollout/activation outcomes across channels/cohorts.

### Konnaxion State (local)
Local operational metadata describing active pack, cache layout, activation history, rollback state.

---

## 4) Kristal artifacts (external, referenced)

kOA uses these terms as **references** to the pinned Kristal v4 dependency:

### Claim-IR
Pre-truth proposed claims produced by extractors.

### Resolved Claim-IR
Pre-truth resolved claims produced by SenTient.

### Validation Report
Deterministic report produced by the validation stage and used as the compilation gate input.

### Kristal Exchange / Exchange Manifest
Canonical truth artifact produced by Kristal compilation.

### Runtime Pack / Runtime Pack Manifest
Derived offline package produced from the Exchange, verified and activated by Konnaxion.

### Render Bundle
Deterministic output artifact produced by Architect-Render containing outputs and trace mapping.

---

## 5) Common mappings and aliases (avoid ambiguity)

### Exchange ID / kristal_id
If both terms appear historically, prefer the Kristal v4 term and treat any alias as legacy.

### Key identifier
Prefer `key_id` (kOA) when describing signature references; treat `kid` as a legacy alias if encountered in older docs.

### Policies
Prefer “policy bundle / pinned policy ref” over ad hoc field names; kOA stores references, Kristal defines canonical manifest structures externally.

---

## 6) Versioning language

### Pinned dependency
A specific, immutable Kristal version (tag/commit) used for validation and conformance.

### Compatibility policy
Rules describing what versions/packs can be activated, including downgrade prevention.

### Legacy compatibility window
A defined period where older spellings/aliases may be accepted on input, but canonical forms are emitted and validated for publication/activation.

---