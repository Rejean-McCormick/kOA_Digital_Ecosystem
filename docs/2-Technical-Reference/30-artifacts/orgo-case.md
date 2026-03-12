# Orgo Case

## Purpose

An **Orgo Case** is the durable, auditable container for governed work in the kOA ecosystem. It groups context, policies, linked artifacts, and a set of Orgo Tasks into a single lifecycle-managed unit.

A Case is the unit operators and automation use to:
- track why work exists,
- enforce stage gating and approvals,
- link work to provenance and outcomes,
- audit decisions and changes over time.

## Scope and ownership

- **Normative here (kOA-owned):** Orgo Case structure, lifecycle states, routing/labeling, audit requirements.
- **Not defined here:** Kristal artifact schemas (Claim-IR, Validation Report, Exchange, Runtime Pack). This doc only references them by content-addressed IDs.

## Core identifiers

- `case_id`: globally unique identifier for the Case.
- `organization_id`: tenant / authority scope.
- `created_at`, `updated_at`: ISO-8601 timestamps.
- `status`: lifecycle state (see below).
- `label`: routing taxonomy string (canonical dot-separated label).

## Required fields (normative)

An Orgo Case MUST include:

- Identity:
  - `case_id`
  - `organization_id`
  - `created_at`
  - `updated_at`

- Classification:
  - `source` (who/what created it: user, automation, system)
  - `type` (broad case type: build, incident, review, ingest, release, etc.)
  - `category` (routing category)
  - `label` (canonical routing label string)

- Lifecycle:
  - `status`
  - `priority` (e.g., P0–P3 or equivalent)
  - `severity` (if applicable)
  - `visibility` (internal/private/public per policy)

- Governance + audit:
  - `owners` (responsible parties)
  - `audit` (append-only event list or references)

- Linkage:
  - `tasks[]` (references to Orgo Tasks)
  - `refs[]` (content-addressed refs to related artifacts, including Kristal artifacts when applicable)

## Optional fields (recommended)

- `summary`: short operator-facing description.
- `description`: longer context / rationale.
- `tags[]`: free-form tags for filtering.
- `policy_context`: references to mandate/policies that govern this case.
- `due_at`: target completion time.
- `sla`: response/resolve targets.

## Lifecycle states (normative)

A Case MUST implement a finite state model with terminal semantics.

Recommended states:

- `OPEN`: created, work not yet complete
- `IN_PROGRESS`: active work underway
- `BLOCKED`: cannot proceed (dependency, policy, external system)
- `WAITING_REVIEW`: awaiting human or policy review/approval
- `COMPLETED`: terminal success (all required tasks satisfied)
- `CANCELLED`: terminal stop (no further work)
- `FAILED`: terminal failure (work ended unsuccessfully)

Rules:

- Terminal states: `COMPLETED`, `CANCELLED`, `FAILED`
- A Case in a terminal state MUST NOT accept new Tasks unless explicitly reopened (policy-controlled).
- Status transitions MUST be recorded in `audit`.

## Tasks and containment

- A Case MUST contain zero or more Orgo Tasks.
- Orgo Tasks reference their parent Case via `case_id`.
- A Case MAY be created before tasks exist (e.g., to reserve audit context), but should eventually contain at least one task or be cancelled.

## References to external artifacts

Cases commonly reference content-addressed artifacts:

- Input snapshot sets (kOA ingest outputs)
- Kristal outputs (Validation Report, Exchange reference, Runtime Pack reference)
- Release records and activation records

Rules:

- References MUST be content-addressed where possible.
- A Case MUST NOT embed large payloads; store pointers and hashes instead.
- References MUST include enough context to resolve the artifact (type + ID + hash/ref).

## Audit model (normative)

A Case MUST provide an append-only audit trail of:

- creation and updates
- status transitions
- task creation/updates (or references)
- policy/approval events
- artifact reference additions/removals
- assignment/ownership changes

Audit entries SHOULD include:
- `event_id`
- `timestamp`
- `actor` (user/service)
- `event_type`
- `diff` or structured fields describing what changed
- optional `reason`

## Label taxonomy (routing)

`label` is a canonical dot-separated string:

`<BASE>.<CATEGORY>.<SUBCATEGORY>.<ROLE...>`

Rules:
- dot-separated, lowercase recommended
- stable segments; changes require a migration plan
- labels SHOULD be derived deterministically from `type/category` and policy routing rules

## Failure modes

- Missing required fields → reject write (fail closed).
- Invalid state transitions → reject write.
- Attempted mutation of audit history → reject write.
- Case references a Task that does not exist → accept only if Task will be created within the same transaction; otherwise reject or mark as inconsistent per policy.

## Schema

The JSON schema for Orgo Case lives at:

- `docs/30-artifacts/schemas/orgo-case.schema.json`

This doc is normative; the schema is the executable contract.