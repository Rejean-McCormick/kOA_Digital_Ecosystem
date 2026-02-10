# Orgo Case & Task Contracts
Path: `docs/04-artifacts-contracts/08-orgo-case-task.md`

## Status
Normative (API-boundary contracts).

## Purpose
Define the canonical **JSON contracts** for **Case** and **Task** at API boundaries, plus the invariants (enums, lifecycles, and deterministic rules) that every service must enforce. :contentReference[oaicite:0]{index=0}

This contract is designed to map cleanly to Orgo’s physical storage (tables `cases` and `tasks`) while keeping the external API naming stable. :contentReference[oaicite:1]{index=1}

---

## Core concepts

### Case
A **Case** is the durable container for an operational situation: it provides context, status, severity, and classification, and can group multiple Tasks. The Case lifecycle is `open → in_progress → resolved → archived` (with allowed re-open from `resolved`). :contentReference[oaicite:2]{index=2}

### Task
A **Task** is the canonical unit of executable work, optionally attached to a Case via `case_id`. Task state changes are governed by a locked state machine and terminal semantics. :contentReference[oaicite:3]{index=3}

---

## Label invariant (classification contract)
Both **Case.label** and **Task.label** use the canonical label format:

`<BASE>.<CATEGORY><SUBCATEGORY>.<HORIZONTAL_ROLE>`

The label is required and is treated as a routing + analytics primitive. :contentReference[oaicite:4]{index=4} :contentReference[oaicite:5]{index=5}

### Broadcast bases (informational by default)
Labels with base `10`, `100`, or `1000` are broadcasts and are **non-actionable by default**: they do not automatically spawn Tasks unless a workflow rule explicitly enables `auto_create_tasks: true` (or equivalent). :contentReference[oaicite:6]{index=6} :contentReference[oaicite:7]{index=7}

---

## Canonical JSON Contract: Case

### Naming and mapping
* External identifier is `case_id` (maps to `cases.id`). :contentReference[oaicite:8]{index=8}

### Schema (YAML)
```yaml
Case:
  type: object
  required:
    - case_id
    - organization_id
    - source_type
    - label
    - title
    - description
    - status
    - severity
  properties:
    case_id:
      type: string
      format: uuid
      description: Stable external identifier (maps from cases.id).
    organization_id:
      type: string
      format: uuid
    source_type:
      type: string
      enum: [email, api, manual, sync]
      description: Origin channel.
    source_reference:
      type: string
      nullable: true
      description: Channel-specific reference (e.g. email message-id, external URI).
    label:
      type: string
      description: Canonical information label "<BASE>.<CATEGORY><SUBCATEGORY>.<HORIZONTAL_ROLE>".
    title:
      type: string
      maxLength: 512
    description:
      type: string
    status:
      type: string
      enum: [open, in_progress, resolved, archived]
      description: Case lifecycle; see "Status lifecycles" section.
    severity:
      type: string
      enum: [minor, moderate, major, critical]
      description: JSON uses lower-case severity tokens; implementations may accept upper-case and normalize.
    reactivity_time:
      type: string
      nullable: true
      description: ISO-8601 duration (e.g. "PT2H"); DB uses interval.
    origin_vertical_level:
      type: integer
      nullable: true
      description: Base part of original label (e.g. 100, 1001).
    origin_role:
      type: string
      nullable: true
      description: Horizontal role of origin (e.g. "Ops.Maintenance").
    tags:
      type: array
      items: { type: string }
      nullable: true
    location:
      type: object
      additionalProperties: true
      nullable: true
      description: Structured location (site, building, GPS, etc.).
    metadata:
      type: object
      additionalProperties: true
      description: Case-level metadata (pattern_sensitivity, review settings, etc.).
    created_at:
      type: string
      format: date-time
      readOnly: true
    updated_at:
      type: string
      format: date-time
      readOnly: true
````

 

### Severity normalization rule

At DB level, Case severity piggybacks on the task severity enum; JSON may use lower-case (`minor|moderate|major|critical`) and must map 1:1 to DB enum values.  

### Minimal example (JSON)

```jsonc
{
  "case_id": "uuid-v4-string",
  "organization_id": "uuid-v4-string",
  "source_type": "email",
  "source_reference": null,
  "label": "1000.12.34.Ops.Maintenance",
  "title": "Wet floor in main corridor",
  "description": "Reported by staff; needs immediate signage and cleanup.",
  "status": "open",
  "severity": "major",
  "reactivity_time": "PT2H",
  "origin_vertical_level": 1000,
  "origin_role": "Ops.Maintenance",
  "tags": ["safety", "wet_floor"],
  "location": { "site": "HQ", "floor": "1" },
  "metadata": { "pattern_sensitivity": "high" },
  "created_at": "2026-02-09T12:00:00Z",
  "updated_at": "2026-02-09T12:00:00Z"
}
```

(Shape aligned to the canonical Case skeleton.) 

---

## Canonical JSON Contract: Task

### Naming and mapping

* External identifier is `task_id` (maps to `tasks.id`).
* Older/legacy docs using `id` must be interpreted as `task_id`. 

### Schema (YAML)

```yaml
Task:
  type: object
  required:
    - task_id
    - organization_id
    - type
    - category
    - label
    - status
    - priority
    - severity
    - visibility
  properties:
    task_id:
      type: string
      format: uuid
      description: Stable external identifier (maps from tasks.id).
    organization_id:
      type: string
      format: uuid
    case_id:
      type: string
      format: uuid
      nullable: true
      description: Case this task belongs to (if any).
    source:
      type: string
      enum: [email, api, manual, sync]
      description: Origin channel.
    type:
      type: string
      description: Domain-level type, e.g. "maintenance", "hr_case", "education_support".
    category:
      type: string
      enum: [request, incident, update, report, distribution]
      description: Global category enum.
    subtype:
      type: string
      nullable: true
      description: Domain-specific subtype (often mirrored into metadata).
    label:
      type: string
      description: Canonical information label.
    title:
      type: string
      maxLength: 512
    description:
      type: string
    status:
      type: string
      enum: [PENDING, IN_PROGRESS, ON_HOLD, COMPLETED, FAILED, ESCALATED, CANCELLED]
      description: TASK_STATUS; JSON may also use lower-case forms that map 1:1.
    priority:
      type: string
      enum: [LOW, MEDIUM, HIGH, CRITICAL]
    severity:
      type: string
      enum: [MINOR, MODERATE, MAJOR, CRITICAL]
    visibility:
      type: string
      enum: [PUBLIC, INTERNAL, RESTRICTED, ANONYMISED]
      description: Governs access and export semantics.
    assignee_role:
      type: string
      nullable: true
      description: Denormalised routing role label (e.g. "Ops.Maintenance").
    created_by_user_id:
      type: string
      format: uuid
      nullable: true
    requester_person_id:
      type: string
      format: uuid
      nullable: true
      description: Person the work is for (student, player, employee, etc.).
    owner_role_id:
      type: string
      format: uuid
      nullable: true
      description: Primary owning role for this task.
    owner_user_id:
      type: string
      format: uuid
      nullable: true
      description: Direct owner; may be null if owned only by a role.
    due_at:
      type: string
      format: date-time
      nullable: true
    reactivity_time:
      type: string
      nullable: true
      description: ISO-8601 duration (e.g. "PT2H").
    reactivity_deadline_at:
      type: string
      format: date-time
      nullable: true
      readOnly: true
      description: Usually created_at + reactivity_time under the active profile.
    escalation_level:
      type: integer
      description: 0 = none; 1+ = depth in escalation path.
    closed_at:
      type: string
      format: date-time
      nullable: true
      readOnly: true
      description: Timestamp when the task entered a terminal state.
    metadata:
      type: object
      additionalProperties: true
      description: Domain-specific fields; must not duplicate core fields.
    created_at:
      type: string
      format: date-time
      readOnly: true
    updated_at:
      type: string
      format: date-time
      readOnly: true
```

 

### Metadata rule (extension point)

`metadata` is the only sanctioned extension surface for domain payload and **must not duplicate core fields**.  

### Minimal example (JSON)

```jsonc
{
  "task_id": "uuid-v4-string",
  "organization_id": "uuid-v4-string",
  "case_id": "uuid-v4-string",
  "source": "email",

  "type": "maintenance",
  "category": "incident",
  "subtype": "cleaning",

  "label": "1000.12.34.Ops.Maintenance",
  "title": "Place signage + clean spill",
  "description": "Deploy wet floor sign and clean corridor.",

  "status": "PENDING",
  "priority": "HIGH",
  "severity": "MAJOR",
  "visibility": "INTERNAL",

  "assignee_role": "Ops.Maintenance",
  "created_by_user_id": null,
  "requester_person_id": null,

  "owner_role_id": null,
  "owner_user_id": null,

  "due_at": "2026-02-09T14:00:00Z",
  "reactivity_time": "PT2H",
  "reactivity_deadline_at": "2026-02-09T14:00:00Z",
  "escalation_level": 0,
  "closed_at": null,

  "metadata": {
    "asset_id": "floor-1-main-corridor",
    "photo_ref": "s3://.../spill.jpg"
  },

  "created_at": "2026-02-09T12:00:00Z",
  "updated_at": "2026-02-09T12:00:00Z"
}
```

(Example fields and semantics aligned to the canonical Task contract.) 

---

## Status lifecycles (deterministic gates)

### Enforcement requirement (all services)

Any service that mutates `cases.status` or `tasks.status` **must enforce** the state machines below; invalid transitions must be rejected and logged as validation errors. 

### Case status lifecycle (CASE_STATUS)

Statuses: `open`, `in_progress`, `resolved`, `archived`. 

Allowed transitions: 

* `open` → `in_progress`
* `open` → `resolved`
* `open` → `archived` (triage out-of-scope/duplicate/spam with reason in metadata)
* `in_progress` → `resolved`
* `in_progress` → `archived` (dropped/invalidated with reason)
* `resolved` → `archived`
* `resolved` → `in_progress` (re-open)

`archived` is terminal in normal flows; reopening is exceptional and must be audited. 

### Task status lifecycle (TASK_STATUS)

Canonical statuses: `PENDING`, `IN_PROGRESS`, `ON_HOLD`, `COMPLETED`, `FAILED`, `ESCALATED`, `CANCELLED`. 

Allowed transitions (locked): 

* `PENDING` → `IN_PROGRESS` | `CANCELLED`
* `IN_PROGRESS` → `ON_HOLD` | `COMPLETED` | `FAILED` | `ESCALATED`
* `ON_HOLD` → `IN_PROGRESS` | `CANCELLED`
* `ESCALATED` → `IN_PROGRESS` | `COMPLETED` | `FAILED`
* `COMPLETED` / `FAILED` / `CANCELLED` are terminal

**Terminal timestamp rule (`closed_at`)**: when entering `COMPLETED`, `FAILED`, or `CANCELLED`, Core Services **must** set `closed_at` to the transition timestamp; otherwise `closed_at` must remain `null`. 

---

## Visibility and comments

* `Task.visibility` uses the locked enum `[PUBLIC, INTERNAL, RESTRICTED, ANONYMISED]`. 
* Task comments use a separate comment visibility enum (`internal_only`, `requester_visible`, `org_wide`); implementations must enforce these values when exposing `task_comments`. 

---

## Compatibility notes

* JSON MAY accept alternative casing for some enums (e.g., lower-case task statuses), but implementations must normalize to the canonical enum set and reject unknown values.  
* Assignment history is not represented by `assignee_role`; it lives in assignment history storage, while `assignee_role` is a denormalized routing/UX convenience. 

