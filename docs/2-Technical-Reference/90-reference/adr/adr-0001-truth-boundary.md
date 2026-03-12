# ADR-0001: Truth Boundary (kOA)

**Status:** Accepted  
**Date:** 2026-02-27  
**Decision Owner:** kOA Architecture  
**Scope:** Ecosystem invariant and operational enforcement (not Kristal schema design)

---

## Context

The kOA ecosystem requires a single, enforceable definition of where “truth” becomes canonical so that downstream systems (distribution, rendering, execution, feedback) cannot accidentally introduce, mutate, or launder unverified claims into canon.

kOA integrates with Kristal as the canonical truth compiler. Kristal v4 defines the normative artifact contracts for Exchange and related Kristal-owned artifacts; kOA must not duplicate those contracts in kOA documentation. kOA must instead enforce the boundary operationally and record evidence.

---

## Decision

1. **Canonical truth exists only after Kristal compilation.**  
   Upstream artifacts (inputs, Claim-IR, Resolved Claim-IR, validation outputs) are non-canonical until the Kristal compile step produces a canonical Exchange artifact.

2. **No compile on fail.**  
   If deterministic validation fails, kOA must stop the build pipeline and must not publish or distribute Exchange/Runtime Pack outputs.

3. **Fail-closed distribution and activation.**  
   Any artifact fetched for activation must be verified (schema compatibility + integrity checks). If verification cannot be completed, activation is rejected and the system rolls back deterministically to the last-known-good state.

4. **Downstream “no new facts.”**  
   Rendering and execution layers cannot introduce new factual claims into canon. They may:
   - trace output to canonical sources,
   - emit governed feedback that creates new work (Cases/Tasks),
   - but must not mutate canonical truth directly.

---

## Consequences

- Orgo must enforce stage order and gates, and record the full evidence chain for each build/release (inputs → validation → canonical outputs).
- Konnaxion must treat verification as a hard gate for activation and rollback.
- Docs and code must treat Kristal schemas as the normative definitions for Kristal artifacts; kOA docs provide only integration and operational policy.

---

## Implementation notes

- The pinned Kristal dependency is recorded under:
  - `docs/40-integration/kristal-v4/pinned-dependency.md`
- kOA operational enforcement points include:
  - `docs/50-operations/pipeline.md`
  - `docs/50-operations/rollback.md`
  - `docs/50-operations/releases.md`

---

## Alternatives considered

- Allow downstream systems to “patch” canon directly (rejected: violates auditability and determinism).
- Duplicate Kristal schemas inside kOA docs (rejected: causes drift and contract divergence).

---

## Links

- `docs/01-principles/02-truth-boundary.md`
- `docs/40-integration/kristal-v4/index.md`