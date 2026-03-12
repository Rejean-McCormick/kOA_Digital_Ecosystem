# Keter: Mandate (Governance Source)

**Normative for kOA:** YES  
**External normative references:** Kristal v4 (pinned) for Kristal artifact contracts; none for Mandate Bundle (kOA-owned)

## Purpose
**Keter (Mandate)** is the ecosystem’s **governance source**: it defines the mission, scope, constraints, and enforceable policy bundles that guide the rest of the system. Keter does not produce canonical truth artifacts; it produces **governance artifacts** that constrain how truth is produced, distributed, rendered, and operationalized.

Keter’s output is consumed by Orgo (pipeline governance), Kristal compilation gates, Konnaxion distribution policy, and Architect behavior policies.

---

## Responsibilities

Keter MUST:
- Define and publish the active **Mandate Bundle** for an organization (mission + constraints + policies).
- Version mandate and policy changes; preserve history (no silent mutation).
- Provide unambiguous policy selections usable by Orgo/Kristal/Konnaxion/Architect.
- Declare enforcement levels (MUST/SHOULD/MAY) and scope of each policy rule.

Keter MUST NOT:
- Directly alter canonical truth artifacts (Exchange) or derived distribution artifacts (Runtime Packs).
- Override deterministic gates at runtime without a versioned mandate/policy change.

---

## Inputs
- Organizational objectives and constraints
- Regulatory/ethics requirements
- Risk appetite and operational SLOs
- Approved policy templates

---

## Outputs

### Primary output: Mandate Bundle (kOA-owned)
A Mandate Bundle is a versioned, auditable container that includes:
- mandate identity and descriptive fields
- constraints and objectives
- one or more policy documents / rule sets
- optional signatures for authenticity

Schema: `docs/30-artifacts/schemas/mandate-bundle.schema.json`

---

## Interfaces

### 1) Retrieve current mandate
- `GET /mandate/current` → Mandate Bundle

### 2) Retrieve by bundle id
- `GET /mandate/{bundle_id}` → Mandate Bundle

### 3) Policy selection interface (to Orgo)
Orgo selects policies by stable identifiers from the Mandate Bundle:
- `policy_id`
- `version`
- optional parameters (must be recorded if used)

Orgo MUST record the selected policy set in operational records (Build/Release), and treat those selections as part of the reproducibility surface.

---

## Invariants (Non-negotiable)

1) **Versioned governance**  
All mandate/policy changes must produce a new bundle version (immutable history).

2) **Deterministic policy interpretation**  
Given the same Mandate Bundle and policy selection, downstream components must interpret policy rules deterministically.

3) **No runtime overrides**  
Runtime behavior changes require a mandate/policy update; ad-hoc overrides are forbidden for anything affecting truth boundary, compile gates, activation gates, or rendering trace requirements.

4) **Scope clarity**  
Policies must declare where they apply (stages, artifacts, components) and how they are enforced.

---

## Failure Modes
- Missing Mandate Bundle (system cannot start governed pipeline safely)
- Ambiguous/conflicting policy rules without deterministic tie-breaking
- Unauthorized mandate updates (signature or access control failure)
- Breaking policy changes without compatibility plan

---

## Observability

Keter MUST emit:
- current mandate bundle id/version
- change events (bundle published/rotated)
- policy selection usage metrics (which policies are active downstream)
- authorization/verification failures

---

## Security
- Mandate Bundle publishing must require strong authentication and authorization.
- Bundles should be signed (recommended) and verified by consumers in strict environments.
- Do not embed secrets in mandate/policy artifacts.

---