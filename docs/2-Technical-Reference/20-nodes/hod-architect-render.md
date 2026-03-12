# Hod: Architect-Render (Deterministic Articulation)

**Normative for kOA:** YES (node specification)  
**External normative references:** Kristal v4 (pinned) for Kristal-facing artifacts/contracts referenced by this node (see `docs/40-integration/kristal-v4/contract-pointers.md`)

## Purpose
Architect-Render turns **validated knowledge** into **user-consumable outputs** (text or structured blocks) under strict rules:

- deterministic output
- **no new facts**
- complete trace coverage (`trace_map`)
- explicit ambiguity handling
- deterministic refusal/error behavior

It is the ecosystem’s articulation layer: formatting and composing, not deciding truth, not executing work.

---

## Non-responsibilities
Architect-Render:
- does not write to Kristal Exchange (canonical truth) or mutate truth artifacts
- does not validate or canonicalize (upstream gate and compile)
- does not execute tasks (SwarmCraft)
- does not decide strategy/plan (Architect-Strategy)
- does not “enrich facts” from the internet or probabilistic sources as truth

---

## Contract boundaries

### Upstream producers
Architect-Render accepts validated bundles produced by:
- Runtime Pack query systems (via Konnaxion/Malkuth), or
- Exchange-derived verified query systems

### Downstream consumers
- Konnaxion UI/navigation surfaces
- Systems persisting render bundles for auditability/publication
- Optional: EkoH surfaces (when presenting ledger-derived signals, without mutating canon)

---

## Responsibilities

### R1 — Accept only validated inputs
Architect-Render MUST operate as a pure function over validated inputs and MUST reject inputs that cannot be proven to derive from a specific Exchange/Pack reference.

### R2 — Deterministic rendering
Given identical validated input bytes + template/profile id+version + language + rendering parameters, the output MUST be identical (including `trace_map`), modulo explicitly allowed non-content metadata excluded by policy.

### R3 — No new facts
Architect-Render MUST NOT introduce factual assertions not supported by validated inputs. It MUST NOT “fill gaps” with plausible guesses.

### R4 — Trace coverage (accountability)
Architect-Render MUST produce a render bundle that includes `trace_map` coverage for factual assertions. If support is missing, it MUST omit, explicitly mark uncertainty (only if upstream uncertainty exists), or refuse deterministically.

### R5 — Ambiguity preservation
If inputs contain unresolved ambiguity, Architect-Render MUST either render ambiguity explicitly or refuse to render the ambiguous claim as a fact. It MUST NOT silently disambiguate unless inputs explicitly declare the resolved selection.

### R6 — Deterministic refusal/error codes
Architect-Render MUST implement stable refusal/error codes and return:
- `status = "ok" | "refused" | "error"`
- `code` (stable string)
- `message` (human-readable)
- optional structured `details`

Minimum refusal/error codes include:
- `UNVERIFIED_INPUT`
- `MISSING_TRACE_IDS`
- `AMBIGUOUS_INPUT`
- `UNSUPPORTED_RENDER_KIND`
- `PROJECTION_MISMATCH`
- `POLICY_VIOLATION_NEW_FACT_RISK`

### R7 — Multilingual constraints
Architect-Render MAY translate labels and explanatory text, but MUST NOT alter factual content. Locale formatting is permitted only if meaning does not change and normalized values remain traceable.

### R8 — Offline correctness / security
Architect-Render MUST NOT require network calls to produce correct factual output. If network calls are used for non-factual assets, they MUST NOT affect factual assertions.

---

## Interfaces

### Inputs (required, conceptual)
Architect-Render MUST accept one of:
1) **Runtime Pack query bundle** (validated, ordered results; references the pack and query contract), or
2) **Exchange-derived verified query bundle** (validated derivation proof + deterministic query spec).

Inputs MUST include stable identifiers sufficient for traceability:
- Exchange/Pack reference
- stable statement identifiers and evidence pointers (preferred `statement_id`)

A separate rendering request MUST declare:
- render kind (article/snippet/summary/qa/card)
- language
- template/profile identifier + version
- constraints (policy-bound; must not change factual content)

### Outputs (required, conceptual)
Architect-Render MUST output a **Render Bundle** as the only permitted boundary artifact.

A Render Bundle MUST include:
- rendered output (text or structured blocks)
- `trace_map` (machine-readable)
- `render_metadata`

Trace map coverage rules:
- every factual statement must have at least one support pointer
- if unsupported: omit, mark uncertainty (only if upstream uncertainty exists), or refuse deterministically

---

## Invariants (must hold)
- Determinism: same validated inputs + same template/profile + same params ⇒ identical output and identical `trace_map`.
- Truth fidelity: no new facts; no external factual enrichment.
- Coverage: every factual statement is supported in `trace_map`.
- Ambiguity: rendered explicitly or refused; never silently coerced.
- Projection consistency: if a projection is declared, render only from that projection and label output metadata.
- Input integrity: reject unverified/untraceable inputs.
- Offline correctness: no network dependency for factual correctness.

---

## Failure modes and required behavior
Architect-Render MUST fail deterministically with stable codes when:
- inputs are unverified / provenance cannot be proven (`UNVERIFIED_INPUT`)
- stable statement/evidence identifiers are missing (`MISSING_TRACE_IDS`)
- ambiguity cannot be rendered under requested constraints (`AMBIGUOUS_INPUT`)
- render kind is unsupported (`UNSUPPORTED_RENDER_KIND`)
- projection constraints are violated (`PROJECTION_MISMATCH`)
- requested output would introduce an unsupported fact (`POLICY_VIOLATION_NEW_FACT_RISK`)

---

## Observability (minimum)
Architect-Render MUST emit:
- Exchange/Pack reference used
- template/profile id + version
- render kind, language, projection
- determinism mode
- status + refusal/error code (if not `ok`)
- trace coverage metrics (e.g., assertions rendered, omitted, marked uncertain, refused)

Recommended metrics:
- deterministic golden-test pass rate (per template/profile)
- refusal rate by code
- coverage gap rate (unsupported assertions encountered)
- latency by render kind and template/profile

---

## Conformance tests (required)
A conformant Architect-Render MUST provide tests for:
- deterministic rendering (golden outputs + golden `trace_map`)
- “no new facts” enforcement (adversarial templates/prompts must not yield unsupported assertions)
- trace coverage enforcement (missing supports ⇒ omit/uncertain/refuse deterministically)
- refusal/error code stability for the minimum code set
- offline correctness (network-disabled run produces identical factual output)