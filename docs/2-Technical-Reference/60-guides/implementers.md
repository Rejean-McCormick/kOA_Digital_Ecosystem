# Implementers Guide (kOA)

**Audience:** engineers implementing kOA components or integrating existing systems into the kOA stage spine.  
**Normative scope:** kOA behavior and integration expectations.  
**External normative dependencies:** Kristal v4 artifact contracts (pinned in `../40-integration/kristal-v4/`).

---

## 1) What you are implementing

kOA is a contract-driven ecosystem with a strict stage spine:

1) Ingest inputs (Orgo)  
2) Extract Claim-IR  
3) Resolve to Resolved Claim-IR (SenTient)  
4) Validate (deterministic gate)  
5) Compile (Kristal) → Exchange + Runtime Pack  
6) Distribute / activate / rollback (Konnaxion)  
7) Render (Architect-Render)  
8) Execute (SwarmCraft / local runtime)  
9) Feedback (governed; creates new work)

Your implementation is conformant if:
- it respects the stage ordering and gates,
- it produces/consumes the correct artifact types at boundaries,
- it is deterministic where required, and
- it is fail-closed where required.

---

## 2) Hard rules (must not violate)

### 2.1 Truth boundary
Only validated + compiled artifacts become canonical. Downstream components must not mutate canon.

### 2.2 No compile on fail
Validation failure must block compilation and publication.

### 2.3 Fail-closed distribution
Activation must verify integrity and compatibility before switching active packs.

### 2.4 Deterministic rendering
Rendering must be deterministic and must not introduce new facts.

---

## 3) Boundary contracts you must treat as external truth

Do not re-implement or “re-interpret” these contracts in kOA docs or code:
- Kristal Exchange contracts
- Runtime Pack contracts
- Canonicalization profiles/versions
- Hash/sign target rules

Instead, depend on the pinned Kristal v4 reference and schemas:
- `../40-integration/kristal-v4/pinned-dependency.md`
- `../40-integration/kristal-v4/contract-pointers.md`

---

## 4) Component implementation notes

### 4.1 Orgo (governance + pipeline)
Implement Orgo as the deterministic controller:
- enforces stage ordering and gates
- records content-addressed references for stage outputs
- emits Build/Release records
- blocks compilation on validation failure

Minimum implementation deliverables:
- stage runner with stable state machine
- artifact store interface (content refs)
- policy selection handling
- audit trail (immutable records, append-only where possible)

### 4.2 Extractors (inputs → Claim-IR)
Extractor outputs are pre-truth:
- must be schema-valid for Claim-IR (as defined in Kristal)
- must be reproducible given the same input snapshot set
- if using probabilistic extraction, pin seeds/models or freeze outputs upstream

### 4.3 SenTient (Claim-IR → Resolved Claim-IR)
Resolution must:
- preserve ambiguity explicitly
- normalize literals deterministically
- produce stable candidate ordering with tie-breakers
- emit stable warnings/errors (codes not free-text)

### 4.4 Validator (deterministic gate)
Validation must:
- accept the same inputs and produce the same Validation Report output
- produce stable error codes and categories
- be fail-closed in Orgo (no compile if validation fails)

### 4.5 Kristal compiler integration
You do not “define” Kristal outputs here. You:
- call the pinned Kristal compiler
- persist outputs and references
- publish artifacts through Orgo’s release workflow

### 4.6 Konnaxion (distribution/activation)
Implement distribution as:
- verify → stage → atomic switch
- deterministic rollback
- downgrade prevention (if policy requires)
- telemetry emission to Orgo

### 4.7 Architect (Strategy vs Render)
- Strategy proposes work (plans/cases/tasks) but does not mutate canon.
- Render produces deterministic bundles with trace coverage.

### 4.8 SwarmCraft / Runtime (execution)
Execution must:
- treat packs as read-only
- record telemetry deterministically (structure stable)
- surface failures with stable codes
- never mutate canon directly (feedback becomes new governed work)

---

## 5) Required records (kOA-owned)

kOA implementations must produce these operational records (schemas are in `../30-artifacts/schemas/`):
- Build Record
- Release Record
- Orgo Case / Task
- (Optional) Konnaxion State / Activation Records (if standardized)

These are kOA-owned and may evolve independently of Kristal, but must not conflict with Kristal artifact contracts.

---

## 6) Determinism checklist

Your component should answer “yes” to:

- Do identical pinned inputs produce identical outputs?
- Is ordering stable and explicitly defined?
- Are failure modes stable (codes/categories) across runs?
- Are all external dependencies either pinned or forbidden in deterministic mode?
- Are seeds/models pinned when probabilistic behavior exists?

If any answer is “no”, the nondeterminism must be moved upstream of the truth boundary and frozen as an input artifact.

---

## 7) Security checklist

- No secret material in logs/telemetry.
- Verify pack integrity before use (fail-closed).
- Sandbox untrusted code/data (especially pack routines).
- Deny network access by default in deterministic modes.
- Enforce quotas/timeouts; prefer explicit policy configuration.

---

## 8) Conformance tests you should run

Minimum tests (see `../40-integration/kristal-v4/conformance.md`):
- deterministic rebuild (same inputs/config/policy → same IDs/hashes)
- fail-closed validation/activation behavior
- deterministic rendering outputs and trace coverage
- rollback determinism

---

## 9) Versioning and compatibility

- Treat Kristal as a pinned dependency; upgrades require an explicit change record and conformance rerun.
- kOA record schemas should use SemVer and be backward compatible where possible.
- Any change affecting gates, determinism, or activation semantics requires an ADR.

---