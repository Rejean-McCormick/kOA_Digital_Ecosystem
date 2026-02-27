# ADR-0008: Orgo Cases, Tasks, and Routing Taxonomy

**Status:** Accepted  
**Date:** 2026-02-09  
**Scope:** Orgo (control plane) — case/task model, routing labels, broadcast semantics

---

## 1. Context

Orgo ingests messy signals (email/API/offline) and converts them into governed work objects. To keep the ecosystem contract-driven and auditable, Orgo must:

- Use one canonical unit of work (Task) across all domains.
- Use Cases as long-lived containers for incidents/patterns/themes.
- Route work deterministically using a single, canonical label grammar that encodes vertical scope and informational intent.
- Treat broadcasts as informational by default (non-actionable unless explicitly configured).
- Convert cyclic “pattern review” into real governed work (Cases/Tasks), not just dashboards.

This ADR records the canonical model and the routing taxonomy that other services, UIs, and domain adapters must align with.

---

## 2. Decision

1) **Task is the canonical unit of work.**  
All domains share the same Task lifecycle, schema, and state machine.

2) **Case is a long-lived container.**  
A Case groups Tasks + context (participants, severity, locations, recurring patterns) and is the unit that cyclic review operates over.

3) **Routing uses one canonical label grammar.**  
Every Case and Task carries exactly one canonical label string:

```text
<BASE>.<CATEGORY><SUBCATEGORY>.<HORIZONTAL_ROLE?>
