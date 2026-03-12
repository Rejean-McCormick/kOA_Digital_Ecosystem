# ADR Index (kOA)

**Purpose:** Architecture Decision Records (ADRs) capture **kOA-owned** decisions: invariants, operational policies, compatibility rules, governance behavior, and component splits.

**Rule:** ADRs must not duplicate Kristal’s normative artifact contracts. When a decision depends on Kristal, the ADR must reference the pinned Kristal v4 dependency (see `docs/40-integration/kristal-v4/pinned-dependency.md`) and point to the specific Kristal doc/schema section.

---

## ADR workflow

### When an ADR is required
Create an ADR when you change:
- kOA invariants (truth boundary, determinism policy, fail-closed rules)
- kOA artifact schemas (Build Record, Release Record, Orgo Case/Task, etc.)
- compatibility or versioning policy
- activation/rollback policy
- component boundaries and responsibilities
- required conformance tests / gates

### Status values
- Draft
- Accepted
- Superseded
- Rejected

### Filename convention
`adr-####-short-slug.md`

---

## ADR list

- `adr-0001-truth-boundary.md`
- `adr-0002-determinism-policy.md`
- `adr-0003-konnaxion-activation-rollback.md`
- `adr-0004-architect-split.md`
- `adr-0005-compatibility-versioning.md`