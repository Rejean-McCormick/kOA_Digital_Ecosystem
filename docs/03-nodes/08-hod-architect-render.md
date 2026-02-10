# Hod (Architect-Render) — Deterministic Articulation / “Form”

## Status
Normative node specification.

## Purpose
Architect-Render turns **validated knowledge** into **user-consumable outputs** (text or structured blocks) under strict rules:

- deterministic output
- **no new facts**
- full trace coverage (`trace_map`)
- explicit ambiguity handling
- deterministic refusal/error behavior

It is the ecosystem’s “articulation layer”: formatting and composing, not deciding truth, not executing work.

---

## Responsibilities

### R1 — Accept only validated inputs
Architect-Render MUST operate as a **pure function** over validated inputs:
- Exchange-derived validated query bundles, or
- Runtime Pack query result bundles

It must reject inputs that are not validation-attested or cannot be traced to a specific `kristal_id` / `pack_id`.

### R2 — Deterministic rendering
Given identical validated input bytes + template/profile id+version + language + rendering parameters, the output MUST be identical (including `trace_map`), modulo explicitly-declared non-deterministic metadata (e.g., timestamps excluded from hashing).

### R3 — No new facts
Architect-Render MUST NOT introduce factual assertions not supported by input claims/statements.
It MAY add structural text and non-factual phrasing, and MAY express uncertainty only if uncertainty exists in the input.

### R4 — Trace coverage (accountability)
Architect-Render MUST output a **render bundle** that includes:
- `rendered_text` (or structured blocks)
- `trace_map` (machine-readable)
- `render_metadata`

Every factual statement must have support pointers in the `trace_map`. If support is missing, the renderer must omit the claim, render it explicitly as uncertainty (only if the uncertainty exists in input), or fail deterministically.

### R5 — Ambiguity preservation
If input contains unresolved ambiguity, Architect-Render MUST either:
- render the ambiguity explicitly, or
- refuse to render the ambiguous claim as a fact and record a refusal reason

It MUST NOT silently pick a disambiguation unless the input explicitly declares a resolved selection.

### R6 — Deterministic refusal/error codes
Architect-Render MUST implement stable refusal/error codes and return:
- `status = "ok" | "refused" | "error"`
- `code` (stable string)
- `message` (human-readable)
- optional structured `details`

### R7 — Multilingual constraints
Architect-Render MAY translate labels and explanatory text, but MUST NOT alter factual content.
Locale-specific formatting (numbers/dates) is allowed only if meaning does not change and the underlying normalized value remains traceable.

### R8 — Offline correctness / security
Architect-Render MUST NOT require network calls to produce correct factual output.
If network calls are used for non-factual assets (e.g., fonts/templates), they MUST NOT affect factual assertions.

---

## Non-responsibilities

- Does not write to Kristal Exchange (canonical truth) or mutate truth artifacts.
- Does not validate or canonicalize (that is upstream governance/validation/compile).
- Does not execute tasks (that is SwarmCraft).
- Does not decide strategy/plan (that is Architect-Strategy).
- Does not “enrich facts” from the internet or probabilistic sources as truth.

---

## Contract boundaries

### Upstream producers
- Kristal Exchange / Runtime Pack query systems (validated bundles)
- Requesting services (Konnaxion UI layer, Orgo orchestration layer, internal APIs)

### Downstream consumers
- Konnaxion (delivery/UI, offline packs, user surfaces)
- EkoH surfaces (if presenting trust/ledger outputs)
- Systems that persist render bundles for auditability or publication

---

## Inputs

### I1 — Validated input bundle (mandatory)
Architect-Render MUST accept only:
A) Runtime Pack query result bundle:
- `pack_id` (or pack content ID)
- `pack_manifest` (or reference)
- `query_contract_version`
- `query_results` (ordered)
- optional `result_provenance`

B) Exchange-derived validated query bundle:
- `source_kristal_id`
- `exchange_manifest` (or reference)
- deterministic query spec
- `query_results` (ordered)
- validation proof that results derive from referenced Exchange under a declared query contract

Inputs MUST include stable identifiers to support traceability:
- `kristal_id` or `pack_id`
- stable identifiers for each asserted fact (preferred `statement_id`)
- stable evidence pointers (`evidence_id` / reference pointers)

### I2 — Rendering request object (mandatory)
Must include:
- `render_kind` (e.g., `article`, `snippet`, `summary`, `qa`, `card`)
- `language` (BCP-47)
- optional `audience_profile` (must not change factual content)
- `template_id` (or `render_profile_id`) + version
- optional `projection` (e.g., `full`, `truthy`)
- `constraints` (policy-bound limits for safety/fidelity)

---

## Outputs

### O1 — Render bundle (mandatory)
Must include:
1) `rendered_text` (or structured blocks)
2) `trace_map`
3) `render_metadata`

#### render_metadata (required)
- `render_kind`
- `language`
- `template_id` + `template_version`
- `source_kristal_id` or `pack_id`
- `query_contract_version` (if runtime)
- `projection`
- `build_id` (Architect run identifier)
- `created` timestamp

#### trace_map (required)
- segments with stable ids (span offsets or block ids)
- per-assertion support pointers, including:
  - `statement_id` (preferred) or deterministic statement pointer
  - `evidence_ids[]` (or reference pointers)
  - `source` (Exchange vs Runtime Pack)
- optional `confidence` only if present in input

### O2 — Status & refusal contract (mandatory)
Architect-Render MUST return deterministic status codes and messages. Minimum refusal/error codes include:
- `UNVERIFIED_INPUT`
- `MISSING_TRACE_IDS`
- `AMBIGUOUS_INPUT`
- `UNSUPPORTED_RENDER_KIND`
- `PROJECTION_MISMATCH`
- `POLICY_VIOLATION_NEW_FACT_RISK`

---

## Invariants (must hold)

- Determinism: same validated inputs + same template/profile + same parameters ⇒ identical render bundle.
- Truth fidelity: no new facts; no external factual enrichment.
- Coverage: every factual statement is supported in `trace_map`.
- Ambiguity: rendered explicitly or refused; never silently coerced.
- Projection consistency: if a projection is declared, only render from that projection and label output.
- Input integrity: reject unverified/untraceable inputs.
- Offline correctness: no network dependency for factual correctness.

---

## Failure modes and handling

- Missing validation/provenance ⇒ refuse (`UNVERIFIED_INPUT`)
- Missing stable IDs for traceability ⇒ refuse (`MISSING_TRACE_IDS`)
- Ambiguity incompatible with constraints ⇒ refuse (`AMBIGUOUS_INPUT`)
- Projection mismatch ⇒ refuse/error (`PROJECTION_MISMATCH`)
- Template/profile missing or unknown ⇒ error/refuse (`UNSUPPORTED_RENDER_KIND` or implementation-specific)
- Attempt to output unsupported factual assertions ⇒ refuse (`POLICY_VIOLATION_NEW_FACT_RISK`)
- Any non-deterministic generation mode enabled ⇒ MUST be explicitly declared and must not contaminate “conformant deterministic” runs

---

## Observability (recommended)

Emit structured events for:
- `tenant_id`, `build_id`, `kristal_id/pack_id`, `template_id`, `template_version`, `language`, `render_kind`, `projection`
- `status`, `code`, `refusal_reason` (if refused)
- trace coverage metrics (assertions count, unsupported attempts count)
- determinism check hashes (optional)

---

## Security and privacy considerations

- Treat all inputs/outputs as potentially sensitive (PII in sources; confidential knowledge).
- Never leak raw evidence blobs unless policy allows; prefer reference pointers.
- If signatures/hashes are declared for inputs, verify or rely on verified delivery policy; reject inputs that cannot be proven validated.

---

## Conformance tests (required)
An implementation claiming conformance MUST provide tests for:
- determinism (same input ⇒ identical render bundle)
- trace coverage (every assertion supported)
- no-new-facts (attempted injection ⇒ refused)
- ambiguity (explicitly rendered or refused deterministically)
- projection handling (mismatch ⇒ deterministic refusal/error)
- offline correctness (no network dependency for factual correctness)

---

## Implementation note (non-normative)
Architect-Render may be implemented as a deterministic NLG engine with strict schema inputs and template/profile selection. Any “style injection” or micro-planning must be treated as non-core or explicitly constrained so it does not violate determinism or factuality.
