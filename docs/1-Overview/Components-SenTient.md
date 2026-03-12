# Components — SenTient (Resolution Plane)

**Normative for kOA:** YES  
**External normative reference:** Kristal v4 (pinned) for Claim-IR / Resolved Claim-IR schemas

## Purpose

SenTient is the ecosystem’s **resolution and reconciliation engine**. It takes extracted claim proposals (Claim-IR) and produces **Resolved Claim-IR** by:
- mapping ambiguous surfaces to explicit, canonical identifiers (e.g., entities/properties),
- normalizing literals deterministically (dates, numbers, units, etc.),
- preserving unresolved ambiguity explicitly (never guessing “silently”).

SenTient sits between **Extract** and **Validate** in the stage spine: it makes claims *explicit enough* for deterministic validation and compilation.

## What SenTient owns (and why)

SenTient owns the **deterministic resolution boundary**:
- candidate selection outputs (with stable ordering + tie-breakers),
- literal normalization,
- explicit ambiguity objects (when resolution is not unique),
- structured warnings/errors that downstream gates can use.

This is what turns “probably this thing” into “these are the possible things, ranked, with reasons” (or “no valid resolution”).

## What SenTient does NOT own

SenTient does not:
- validate acceptance (that is the Validation gate),
- compile or mutate canonical truth (Kristal),
- publish artifacts to channels (Orgo/Konnaxion),
- invent missing facts or “fill gaps”.

## Inputs (conceptual)

- **Claim-IR batch** (structured claim proposals; pre-truth)
- **Resolution policy + candidate sources** (pinned versions/resources)
- Optional: provenance attachments for audit (as references)

## Outputs (conceptual)

- **Resolved Claim-IR batch** reference
  - resolved identifiers where possible,
  - normalized literals,
  - explicit ambiguity preserved where unresolved,
  - diagnostics (warnings/errors) emitted deterministically.

On **partial resolution**, SenTient still emits Resolved Claim-IR but marks affected claims as ambiguous/rejected with diagnostics.

## Key invariants

- **Deterministic behavior:** same inputs + same pinned resources/policies ⇒ same outputs.
- **Ambiguity preservation:** unresolved ambiguity must remain explicit (no silent coercion).
- **Stable ordering:** candidate lists must be stably ordered with deterministic tie-breakers.
- **Stable diagnostics:** warnings/errors use stable codes and structured parameters (not free-text-only).

Orgo decides gating rules; SenTient must provide the signals needed for those decisions.

## Error model (high-level)

SenTient emits stable warning/error codes. Typical categories include:
- entity resolution not found / ambiguous,
- property resolution not found / ambiguous,
- literal normalization invalid / lossy,
- evidence pointer invalid,
- policy violation,
- missing pinned resource/version,
- internal resolver error.

Each diagnostic should include: claim reference, field/path (when relevant), severity, and deterministic message template + parameters.

## Failure modes (examples)

- Candidate source unavailable or version not pinned
- Ambiguity too high (no deterministic selection)
- Invalid or lossy literal normalization
- Policy blocks a resolution path
- Resource integrity issue (hash/signature mismatch)
- Determinism violation (non-stable ordering, non-pinned dependency)

## Observability (minimum)

SenTient should emit:
- resolver identity/version/config reference
- input content reference(s)
- pinned policy/resource versions
- resolved-claim-ir output reference
- diagnostic code counts + key rates

Suggested metrics:
- throughput (claims/sec)
- latency (p50/p95/p99)
- candidate hit-rate (entity/property)
- ambiguity rate
- rejection rate
- normalization error rate
- resource cache hit-rate

## Security and privacy

- Treat inputs as potentially sensitive; follow tenant policy for logging/redaction.
- Resolver resources must be integrity-checked and version-pinned.
- No external network calls unless explicitly allowed by policy and audited.

## Conformance expectations (summary)

Minimum tests typically cover:
- deterministic rerun (same inputs/resources ⇒ same outputs)
- ambiguity preservation
- literal normalization golden tests
- resource pinning failures (mismatched resource ref ⇒ failure)
- diagnostic stability (codes and shapes stable across patch releases)

## Related pages

- [Lifecycle](Lifecycle)
- [Integration — Kristal v4](Integration-Kristal-v4)
- [Operations](Operations)