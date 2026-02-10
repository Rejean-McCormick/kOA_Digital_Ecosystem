# Global Invariants (System Laws)

**File:** `docs/01-principles/01-invariants.md`  
**Status:** Canonical (normative)  
**Purpose:** Define the non-negotiable rules that make the ecosystem auditable, reproducible, and stable across implementations.

---

## Definitions

- **Artifact:** A typed payload crossing a boundary (e.g., Claim-IR, Resolved Claim-IR, Validation Report, Exchange, Runtime Pack, Render Bundle).
- **Gate:** A deterministic acceptance point; failure blocks downstream stages (fail-closed).
- **Canonical Truth:** Truth that exists only after validation and compilation into Exchange (content-addressed, immutable).
- **Governed Change:** Feedback triggers new work (Cases/Tasks) and new artifacts; no in-place mutation of canonical truth.
- **Fail-closed:** If declared integrity material fails verification, operations must stop (no best-effort).
- **Determinism:** Same inputs + config + policy selections must yield identical outputs (or canonical-identical outputs) in deterministic modes.

---

## INV-001 — Mandate required (system must not operate without it)

**Rule:** If the mandate/policy context is absent or revoked, the system MUST NOT operate.  
**Enforced by:** Orgo admission rules; deployment policy.  
**Tests:** Start workflow without mandate → refused.

---

## INV-002 — Preserve provenance; do not mutate raw sources

**Rule:** Input sources are not “truthed” at ingestion; provenance MUST be preserved; raw sources/snapshots MUST NOT be mutated.  
**Enforced by:** Ingestion adapters; storage policy.  
**Tests:** Attempt to rewrite a source snapshot → forbidden.

---

## INV-003 — Blueprint is deterministic and versioned

**Rule:** Schemas/config/policies MUST be deterministic, versioned, and changed only by explicit revision. Builds MUST pin the Blueprint bundle identity (hash/version).  
**Enforced by:** Blueprint registry; build manifests.  
**Tests:** Build with unknown/unpinned schema revision → refused.

---

## INV-004 — Truth boundary: canonical truth exists only after validate + compile

**Rule:** Canonical truth is created only after deterministic validation and compilation into Exchange.  
**Enforced by:** Orgo stage gating; Kristal compiler contract.  
**Tests:** Attempt to publish without validation stage → refused.

---

## INV-005 — Canonical truth is immutable

**Rule:** Once compiled/published as Exchange, canonical truth is immutable; updates MUST produce new artifacts/versions (no in-place mutation).  
**Enforced by:** Content-addressed IDs; write permissions; storage immutability.  
**Tests:** Attempt in-place edit of Exchange → rejected.

---

## INV-006 — No compile on fail (hard gate)

**Rule:** If validation fails, compilation MUST NOT proceed (Exchange/Runtime Pack MUST NOT be produced).  
**Enforced by:** Orgo workflow contract; pipeline controller.  
**Tests:** Inject failing validation report → verify no compile stage executed.

---

## INV-007 — Strict pipeline boundaries (proposal → resolution → acceptance)

**Rule:**
- Extractors MUST output **Claim-IR only** (schema-constrained proposals with evidence + uncertainty).
- Resolution MUST output schema-valid **Resolved Claim-IR** with explicit ambiguity (no forced disambiguation).
- Validation MUST be deterministic under pinned policy/config and MUST block compilation on failure.

**Enforced by:** Schema validation at each boundary; Orgo gating.  
**Tests:** Non-Claim-IR extractor output → rejected; ambiguous entity resolution → remains explicit.

---

## INV-008 — Deterministic reproducibility (identity + manifests)

**Rule:** Every Exchange and Runtime Pack MUST be accompanied by reproducibility metadata sufficient to re-derive outputs:
- pinned input snapshots
- blueprint hash
- policy_id / policy selections
- canonicalization profile
- component identity/version + config hash
- declared hashes/signatures

Under identical pinned conditions, outputs MUST be identical (or canonical-identical) and hashes/IDs MUST match.

**Enforced by:** Compiler; Orgo build records; CI gates.  
**Tests:** Rebuild same snapshots/config → identical Exchange hash/ID and Pack hash/ID.

---

## INV-009 — Fail-closed verification for integrity material

**Rule:** If an artifact declares hashes/signatures/key refs, verifiers MUST fail-closed on any mismatch or malformed integrity material.  
**Enforced by:** Orgo pre-publish checks; Konnaxion activation checks.  
**Tests:** Tamper payload → activation blocked.

---

## INV-010 — Distribution safety: atomic activation + deterministic rollback

**Rule:** Runtime Pack activation MUST be atomic; rollback MUST exist and MUST be deterministic given the same trigger sequence and available packs (pinned or last-known-good).  
**Enforced by:** Konnaxion distribution facet.  
**Tests:** Mid-activation failure → no partial state; rollback restores known-good.

---

## INV-011 — No new facts downstream (rendering is non-creative)

**Rule:** Rendering MUST NOT introduce unsupported facts. Render outputs MUST be deterministic under pinned inputs/template/params and MUST provide trace coverage in the Render Bundle `trace_map`. If trace coverage is insufficient, the renderer MUST deterministically omit, mark uncertain (only when upstream uncertainty exists), or refuse per policy.  
**Enforced by:** Architect-Render contract; Render Bundle schema + conformance tests.  
**Tests:** Inject unsupported statement → refused (strict policy); missing trace support → omit/uncertain/refuse deterministically.

---

## INV-012 — Feedback is governed and non-mutating to Exchange

**Rule:** Feedback/telemetry MUST NOT mutate Exchange. It MUST become governed work (new Cases/Tasks) that can trigger new pipeline runs producing new artifacts.  
**Enforced by:** Orgo workflow policy; Konnaxion signal handling.  
**Tests:** Attempt direct Exchange mutation from feedback → rejected + governed work created.

---

## INV-013 — End-to-end auditability (evidence + provenance + correlation)

**Rule:** Every output MUST be traceable to evidence and workflow provenance:
- Orgo MUST produce Build Records with full input/config/output linkage
- Render Bundles MUST include `trace_map` for factual content
- Correlation identifiers (request/build/workflow/task/case) SHOULD be propagated end-to-end

**Enforced by:** Orgo logging; build manifests; Render Bundle trace_map.  
**Tests:** For any rendered factual assertion, produce a trace chain back to validated lineage and input snapshots.

---

## INV-014 — Tenant isolation and tenant-scoped trust roots

**Rule:** In multi-tenant deployments, data MUST be isolated and verification MUST use the correct tenant/channel trust roots.  
**Enforced by:** Orgo tenancy boundaries; Konnaxion trust-root pinning.  
**Tests:** Cross-tenant artifact access → blocked; wrong trust root → verification fails closed.

---

## INV-015 — Work must be typed (Cases/Tasks) and telemetry is mandatory

**Rule:** Execution/control flows MUST be expressed as typed work objects (Cases/Tasks) with lifecycle enforcement. Execution layers MUST emit mandatory telemetry (correlation IDs + state transitions + produced artifacts).  
**Enforced by:** Orgo task engine; SwarmCraft execution loop (or equivalent).  
**Tests:** Untyped command injection → rejected; missing telemetry → fails compliance.

---

## Minimal negative tests (required)

- New factual statement without trace support → must omit/uncertain/refuse deterministically (strict policy: refuse).
- Tampered Runtime Pack → must fail-closed on verification.
- Attempt direct mutation of Exchange from feedback → must reject; must open governed work.
