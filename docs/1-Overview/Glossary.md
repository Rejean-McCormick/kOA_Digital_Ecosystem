# Glossary

## Activation
Making a specific Runtime Pack the currently active pack for a target scope (environment/channel/cohort), typically as an atomic switch.

## Artifact
A versioned, typed output that crosses a component boundary (inputs/outputs/results/evidence). Artifacts are used to make runs reproducible and auditable.

## Audit trail
The recorded evidence showing what ran, what was produced, what was released, where it went, and why decisions were made.

## Blueprint
An auditable plan for a run that declares required inputs, gates, determinism constraints, and rollback expectations.

## Build
A completed pipeline run that produced eligible outputs (e.g., a Runtime Pack) and recorded evidence about inputs, policies, gates, and results.

## Canonical truth
The system’s authoritative, compiled knowledge state. It exists only after deterministic validation and compilation.

## Channel
A named release lane (e.g., canary/stable/lts) that defines rollout behavior and guardrails.

## Claim
A proposed statement about the world (an assertion) produced by extraction, before validation/compilation.

## Claim-IR
A structured representation of claims intended for validation/compilation. Pre-truth.

## Cohort
A subset of a channel (by tenant group, region, rollout percentage, etc.) used to ramp exposure gradually.

## Compile / Compilation
Transforming validated knowledge into canonical truth artifacts and derived runtime artifacts (e.g., Runtime Packs).

## Conformance
Meeting the required integration and safety expectations (e.g., pinned dependencies, schema validation, fail-closed behavior, determinism).

## Determinism
The property that the same pinned inputs, policies, and configuration produce the same outputs (or canonically identical outputs).

## Distribution
Making a Runtime Pack available to runtime systems (fetch/cache) without implying activation.

## Exchange
A standardized bundle of artifacts used to move validated/compiled knowledge across boundaries (as defined by Kristal).

## Fail-closed
If a required check cannot be completed or fails, the system must not proceed (e.g., no activation without verification).

## Feedback
Signals or observations produced downstream (runtime/rendering/execution) that are turned into new governed work, not direct truth mutation.

## Gate
A pass/fail checkpoint that must succeed before moving to the next stage (e.g., validation gate, verification gate).

## Ingest
Capturing inputs as immutable, provenance-linked snapshots so they can be audited and replayed.

## Integrity check
Verification that an artifact/pack is unmodified and complete (e.g., hash/signature/manifest checks).

## Kristal
The truth compilation subsystem and the normative source for Kristal artifact contracts (schemas/canonicalization/signing rules).

## Konnaxion
The distribution/runtime platform that fetches/caches packs, verifies them fail-closed, activates atomically, and supports deterministic rollback.

## Last-known-good (LKG)
A known safe pack/version that can be selected deterministically as a rollback target.

## Mandate Bundle
The governance and policy context used to run and release safely (rules, constraints, approvals, and references).

## Malkuth
The runtime environment that serves queries and execution over the currently active pack in a safe, repeatable way.

## Node
A named component with a clear responsibility and defined artifact boundaries.

## Orgo
The control plane: orchestrates stage ordering, enforces gates, records operational evidence, and drives releases.

## Pack / Runtime Pack
A portable, offline-capable bundle of compiled knowledge and required runtime metadata used by runtime systems.

## Pinning / Version pinning
Locking dependencies, policies, or packs to a specific version/reference to prevent “floating latest” behavior.

## Policy
The rules and constraints that govern what is allowed (validation requirements, rollout rules, downgrade prevention, logging/redaction, etc.).

## Provenance
The recorded origin and lineage of inputs and outputs (what source data was used, how it was processed, and by which versions).

## Renderer
A component that produces user-facing outputs deterministically from the active pack and must not introduce new facts.

## Resolution
The process of mapping ambiguous surfaces to explicit identifiers and normalized literals, preserving ambiguity explicitly when unresolved.

## Resolved Claim-IR
Claim-IR after resolution: explicit identifiers/normalized literals plus explicit ambiguity and diagnostics as needed.

## Release
The operational act of distributing and activating a specific Runtime Pack in target environments/channels/cohorts.

## Rollback
Restoring a previous pack/version (typically LKG or pinned prior) via a controlled, atomic activation when a release fails or regresses.

## Schema
A formal contract describing artifact structure and validation requirements.

## Stage spine
The end-to-end lifecycle of stages from ingest through feedback, used as the shared mental model for the system.

## Telemetry
Operational signals emitted by components (metrics/logs/traces/events) used for monitoring, debugging, and audit support.

## Tenant isolation
Ensuring one tenant’s data, behavior, and failures cannot affect another tenant.

## Trace
A record linking a user-facing output back to the canonical sources/artifacts it was derived from (for transparency and auditability).

## Validate / Validation
Deterministic acceptance/rejection of resolved claims against rules and constraints, producing a validation report.

## Verification
Runtime-side checks (integrity/compatibility/policy) that must pass before activation is allowed.