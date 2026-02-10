# Keter — Mandate (Why / Policy Root)

## Summary
Keter is the root mandate of the ecosystem: the “Why” that authorizes operation and defines ethical scope, objectives, and success criteria. Keter is an external input (human + configuration) and is treated as read-only during runtime.

If Keter is absent, revoked, or null, the ecosystem must not operate.

---

## Responsibilities
- Define the system’s purpose and non-negotiable intent (“Why”).
- Declare ethical scope, allowed/forbidden action boundaries, and risk posture.
- Declare success criteria (what counts as “done” / “good” / “failure”).
- Provide the top-level policy context consumed by all nodes.
- Provide the top-level alignment constraint for planning, rendering, distribution, and execution.

---

## Contract Boundaries
### Position in the system
- Keter has no upstream producer inside the ecosystem.
- Keter is supplied by a human sponsor / author / owner as a configuration artifact.
- Downstream nodes may reference Keter but MUST NOT modify it.

### Trust model
- Keter is treated as authoritative for the current system instance.
- Any change to Keter is a governed reconfiguration (new mandate version / new system instance context).

---

## Inputs
- Human mandate / charter / policy bundle (versioned).
- Optional: governance metadata (owner identity, review/approval signatures, expiration, jurisdiction tags).

---

## Outputs
- Read-only “policy context” available to all nodes at runtime:
  - mandate_id / mandate_version
  - scope constraints
  - ethical constraints
  - success criteria
  - permitted capabilities
  - forbidden actions
  - risk thresholds / safety posture
  - audience / domain boundaries (if relevant)

---

## Primary Artifact: Mandate Bundle (recommended)
**Status:** required conceptually; formal schema should be standardized.

Suggested minimal structure:
- identity:
  - mandate_id (stable)
  - version
  - created_at
  - owner / issuer
- intent:
  - mission_statement
  - objectives (ranked)
  - non_goals
- ethics & safety:
  - prohibited_domains
  - prohibited_actions
  - red_lines (hard stops)
  - risk_profile (low/medium/high + rationale)
- scope:
  - allowed_sources / forbidden_sources (optional)
  - allowed_outputs / forbidden_outputs (optional)
  - jurisdiction/time constraints (optional)
- success criteria:
  - measurable outcomes (KPIs)
  - quality thresholds
  - acceptance conditions
- operational constraints:
  - determinism posture (strict/relaxed by subsystem)
  - audit/retention requirements
  - required traceability level

---

## Invariants (hard)
1) **Runtime immutability**
   - The active mandate is read-only during runtime.
2) **No mandate, no operation**
   - If mandate is absent, revoked, or null ⇒ the system must stop, idle, or refuse work.
3) **Downstream alignment**
   - No node may produce outputs or actions that violate Keter constraints.
4) **Governed change**
   - Mandate changes produce a new version / new context; never silent mutation.

---

## Enforcement Points (where Keter is checked)
- Orgo (control plane)
  - blocks creation/execution of work items that conflict with scope or prohibitions.
  - blocks pipeline initiation when mandate is missing.
- SenTient (resolution)
  - may refuse to resolve into actionable decisions if mandate forbids the domain or intent.
- Architect-Strategy (planning)
  - may not propose tasks outside scope; must prioritize according to objectives.
- Architect-Render (articulation)
  - must refuse or constrain outputs that violate ethical/scope constraints.
- Konnaxion (distribution)
  - may refuse activation/distribution of packs/outputs when mandate constraints are not satisfied.
- SwarmCraft (execution)
  - must refuse execution of tasks that violate prohibitions or jurisdiction constraints.

---

## Failure Modes
- MANDATE_MISSING
  - No active mandate bundle found.
  - Expected behavior: refuse all work; system idle/stop.
- MANDATE_REVOKED / EXPIRED
  - Mandate exists but is no longer valid.
  - Expected behavior: refuse new work; halt pipelines; optionally revoke distributions.
- MANDATE_CONFLICT
  - Proposed plan/task/output violates scope, ethics, or prohibited action list.
  - Expected behavior: deterministic refusal with reason code + trace to violated rule.
- MANDATE_SCHEMA_INVALID
  - Bundle fails schema validation.
  - Expected behavior: reject mandate; do not operate.

---

## Observability
Minimum telemetry emitted by every node that consumes Keter:
- mandate_id, mandate_version (in every log line / event)
- check_result: pass/fail
- failure_code (if fail)
- violated_rule_id (if fail)
- correlation_id linking the check to a Case/Task/build/render event

Optional:
- mandate_coverage metrics:
  - % of operations with explicit mandate checks
  - top violated rules
  - refusal rate by rule category

---

## Interfaces
### Read API (conceptual)
- `GET /mandate/active` → returns active mandate bundle + version metadata
- `GET /mandate/{mandate_id}/{version}` → returns immutable historic versions (if retained)

### Injection / update (governed, not runtime)
- `SET_MANDATE` is not a runtime endpoint; it is a governed operation:
  - requires explicit approval/authorization
  - produces a new version
  - triggers controlled restart/rebuild of dependent caches/manifests

---

## Conformance Tests (minimum)
1) Null mandate ⇒ system refuses to run (no build, no task execution, no publish).
2) Mandate forbids domain X ⇒ any plan/render attempting X deterministically refuses.
3) Mandate change ⇒ version increments; old version remains retrievable (if retention enabled).
4) Every emitted artifact/output references mandate_id/version.

---

## Open Questions / TODO
- Formalize `mandate-bundle.schema.json` and stable rule IDs for prohibitions.
- Specify the exact rule language (enums vs expression DSL).
- Specify whether mandate is embedded into:
  - build records (Orgo),
  - Exchange metadata (Kristal),
  - render bundles (Architect),
  - distribution manifests (Konnaxion).
