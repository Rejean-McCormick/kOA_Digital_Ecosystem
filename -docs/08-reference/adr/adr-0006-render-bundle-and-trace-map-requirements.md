# ADR-0006: Render Bundle and Trace Map Requirements

- **Status:** Accepted
- **Date:** 2026-02-09
- **Owners:** Architect / Rendering Working Group
- **Supersedes:** None
- **Superseded by:** None

---

## Context

The ecosystem requires user-facing outputs (reports, pages, summaries, answers, briefings) that:

- cannot introduce new facts beyond validated canon
- remain reproducible (deterministic) for a fixed set of inputs and parameters
- support auditability: every factual assertion must be traceable back through the governed pipeline
- are portable across renderers and toolchains (typed artifact, schema-defined)

Without an explicit rendering contract, “presentation” can become an informal truth engine, causing drift and breaking the truth boundary.

---

## Decision

Define a canonical **Render Bundle** artifact produced by **Architect-Render** with mandatory **trace_map** semantics and enforcement rules.

### 1) Introduce Render Bundle as a typed artifact
A Render Bundle is a versioned, schema-defined output package containing:

- a stable identifier (MUST match the Render Bundle schema; e.g., `bundle_id` if that is the schema’s normative field)
- `renderer` identity and version
- input references (Kristal Exchange and/or Runtime Pack plus governed lineage refs)
- output payloads (text/HTML/Markdown/JSON, etc.)
- `trace_map` (mandatory for factual outputs)
- policy/config references (blueprint hash, policy_id, determinism mode)
- deterministic codes for refusals/omissions/uncertainty markings

Render Bundle is the **only** permitted output format for Architect-Render at system boundaries.

### 2) Make trace_map mandatory for factual assertions
For any output that contains factual assertions, Architect-Render MUST produce `trace_map` entries that map each assertion to one or more **support pointers**.

Support pointers MUST reference validated lineage artifacts:

- Input Snapshot → Claim-IR → Resolved Claim-IR → Validation Report → Exchange (and/or Runtime Pack)

Trace MUST be expressible as stable identifiers (content refs + offsets or stable claim IDs / record IDs).

### 3) Enforce “No new facts downstream” as a hard rendering constraint
Architect-Render MUST NOT introduce factual assertions that lack trace coverage.

If a candidate statement lacks trace coverage, Architect-Render MUST choose one of the following behaviors (policy-defined, but deterministic):

- **OMIT**: exclude the statement
- **MARK_UNCERTAIN**: only if uncertainty exists upstream (e.g., UNRESOLVED / RESOLVED_MULTI)
- **REFUSE**: fail the render deterministically with a structured refusal code

Architect-Render MUST NOT “fill gaps” with plausible guesses.

### 4) Require deterministic rendering
Given the same:
- Exchange/Pack reference(s)
- blueprint hash and policy_id
- template/version (if templates are used)
- parameters
- locale/language selection rules
- determinism mode

Architect-Render MUST produce identical output (byte-identical or canonical-identical) and identical trace_map, except for allowed non-content metadata fields explicitly excluded by policy (e.g., timestamps in metadata, never in hashed output payload).

### 5) Separate Strategy from Render
Architect is formally split into two contracts:

- **Architect-Strategy**: planning/orchestration (may propose work)
- **Architect-Render**: deterministic articulation (must obey trace/no-new-facts)

Strategy outputs do not count as canonical user-facing truth; only Render Bundles do.

---

## Render Bundle requirements (normative)

### A) Minimal required fields
A conforming Render Bundle MUST include:

- `bundle_version` (semantic version)
- stable identifier (MUST match schema; e.g., `bundle_id`)
- `renderer` `{ component, version, instance_id? }`
- `inputs`:
  - `exchange_ref` OR `runtime_pack_ref` (at least one)
  - `blueprint_hash`
  - `policy_id`
  - `pipeline_refs` (refs to Resolved Claim-IR + Validation Report, at minimum)
- `output`:
  - `format` (e.g., markdown/html/json)
  - `content` or `content_ref` (content-addressed preferred)
- `trace_map` (required if output contains any factual assertions)
- `determinism` declaration:
  - canonicalization profile (if applicable)
  - stable ordering declaration
- `events`:
  - `omissions[]` (if any)
  - `refusals[]` (if refused or partially refused)
  - `uncertainties[]` (if uncertainties are surfaced)

### B) Trace map structure
`trace_map` MUST support mapping from output segments to supports.

Minimum shape (conceptual):

- `trace_map.version`
- `trace_map.coverage` summary:
  - `assertions_total`
  - `assertions_traced`
  - `assertions_omitted`
  - `assertions_uncertain`
  - `assertions_refused`
- `trace_map.entries[]` where each entry contains:
  - `output_span` (start/end offsets or stable anchor IDs)
  - `assertion_id` (stable ID or hash of assertion canonical form)
  - `supports[]` (one or more):
    - `exchange_ref` + `record_id` (preferred)
    - or `resolved_claim_ref` + `claim_id`
    - plus optional evidence pointers back to input snapshots
  - `support_strength` (optional; informative only)
  - `state`:
    - `TRACED` | `UNCERTAIN` | `OMITTED` | `REFUSED`

### C) What counts as a “factual assertion”
A factual assertion includes any statement presented as truth about:
- entities, attributes, relationships
- dates, quantities, counts
- causality or categorical membership (unless explicitly hedged as uncertain)

Purely procedural text (“this report covers…”) does not require trace_map coverage, but any factual content does.

### D) Handling ambiguity
If upstream resolution is ambiguous:
- render MAY present multiple candidates as alternatives (with explicit ambiguity)
- render MAY present uncertainty language, but MUST link it to upstream uncertainty states
- render MUST NOT collapse ambiguity into a single binding without validated support

### E) Refusal/omission determinism
Refusal and omission decisions MUST be deterministic under policy:
- same inputs → same omit/refuse behavior
- refusal includes deterministic code + reason category
- omission includes deterministic omission codes and references to missing supports

---

## Rationale

- **Trace map** makes the truth boundary enforceable in the presentation layer.
- **Deterministic rendering** enables reproducible builds, caching, offline parity, and audit.
- **No new facts** prevents the renderer from becoming an informal canonicalizer.
- **Typed Render Bundle** provides a portable, toolchain-neutral artifact for distribution, review, and verification.
- **Explicit uncertainty behavior** preserves ambiguity instead of silently coercing.

---

## Consequences

### Positive
- Outputs become audit-grade: every factual statement is accountable.
- Rendering becomes safe to distribute and cache (offline and online).
- Clear integration boundary for UIs and external consumers.
- Enables conformance testing for hallucination drift at the contract level.

### Costs / Trade-offs
- Renderer complexity increases (must maintain trace_map and enforce constraints).
- Some outputs will be shorter or refused when canon coverage is insufficient.
- Requires stable IDs and canonicalization conventions for mapping and hashing.

---

## Alternatives considered

1) **No trace_map, rely on “best effort citations”**
- Rejected: not enforceable, not testable, high drift risk.

2) **Allow render to create “soft facts”**
- Rejected: violates truth boundary and encourages canon contamination.

3) **Embed trace only as human footnotes**
- Rejected: machine verification becomes unreliable; coverage cannot be tested.

---

## Follow-ups

- Lock `render-bundle.schema.json` (artifact schema) and `trace_map` formal types.
- Normalize naming across docs and schema:
  - identifier field (`render_id` vs `bundle_id`)
  - required input reference field names
- Define canonical anchoring mechanism (offsets vs anchors) for stable cross-format trace.
- Define policy profiles for omit/uncertain/refuse behavior.
- Add conformance tests:
  - deterministic rendering golden tests
  - trace coverage enforcement tests
  - adversarial prompt/template tests to ensure “no new facts”

---

## Links

- `docs/04-artifacts-contracts/07-render-bundle.md`
- `docs/03-nodes/08-hod-architect-render.md`
- `docs/01-principles/01-invariants.md`
- `docs/07-implementation-guides/03-testing-conformance.md`
