# Architect (Articulation Plane)

Architect is the ecosystem’s **articulation layer**: it turns canonical knowledge into **actionable plans** and **user-facing outputs** while preserving the truth boundary.

Architect is explicitly split into two roles:

- **Architect-Strategy (Netzach):** decides *what to do next* (governed work proposals)
- **Architect-Render (Hod):** explains *what is true* (deterministic outputs with traceability)

---

## Why Architect is split

Combining planning and rendering creates failure modes (plans being treated as truth, rendering varying based on planning heuristics, feedback loops hiding provenance). The split enforces a clean separation between **governed proposals** and **deterministic articulation**.

---

## Architect-Strategy (Netzach)

### What it does
Architect-Strategy converts **canonical truth + operating context** into **governed work proposals** that Orgo can accept, decompose into Cases/Tasks, execute, and audit.

### What it produces
- Plans that map cleanly to **Orgo Cases/Tasks**
- (Optionally) rendering specs/templates selection (without inventing facts)

### Key constraints (plain language)
- Strategy proposals are **not truth**. They can recommend actions, but they do not publish or mutate canonical knowledge.
- Strategy may use heuristic/probabilistic methods, but outcomes must be captured as governed work objects.

---

## Architect-Render (Hod)

### What it does
Architect-Render turns **validated knowledge** into **user-consumable outputs** (text or structured blocks) under strict rules:
- deterministic output
- **no new facts**
- complete trace coverage (`trace_map`)
- explicit ambiguity handling
- deterministic refusal/error behavior

Architect-Render is formatting and composing—not deciding truth, not executing work.

### What it consumes (conceptual)
Architect-Render accepts only **validated** bundles derived from:
- Runtime Pack query results, or
- Exchange-derived verified query results
plus a **pinned** template/profile and rendering parameters.

### What it produces (conceptual)
A **Render Bundle** containing:
- the rendered output
- a machine-readable `trace_map`
- render metadata (template/profile version, language, etc.)

### Non-negotiable behavior
- **Accept only validated inputs** (reject unverified/untraceable provenance)
- **Deterministic rendering** (same pinned inputs + same template/profile/params ⇒ identical output and identical `trace_map`)
- **No new facts** (no “gap filling” with plausible guesses; no external factual enrichment)
- **Trace coverage** (every factual statement must trace to validated lineage; otherwise omit / mark-uncertain only if upstream uncertainty exists / or refuse)
- **Ambiguity preservation** (render ambiguity explicitly or refuse; never silently disambiguate)
- **Offline correctness** (no network dependency for factual correctness; network calls, if used for non-factual assets, must not affect factual assertions)

### Deterministic refusal (examples)
Architect-Render refuses deterministically with stable codes when, for example:
- input provenance cannot be proven (`UNVERIFIED_INPUT`)
- stable evidence identifiers are missing (`MISSING_TRACE_IDS`)
- ambiguity cannot be rendered under requested constraints (`AMBIGUOUS_INPUT`)
- render kind is unsupported (`UNSUPPORTED_RENDER_KIND`)
- projection constraints are violated (`PROJECTION_MISMATCH`)
- requested output would introduce an unsupported fact (`POLICY_VIOLATION_NEW_FACT_RISK`)

---

## Where Architect sits in the lifecycle

- **Render is Stage 7**: it runs after validation + compilation and produces Render Bundles with trace coverage.
- **Strategy feeds Orgo**: it produces governed work proposals that Orgo manages through its lifecycle and auditing.

---

## Operational signals (what operators should expect)

Architect-Render should emit, at minimum:
- Exchange/Pack reference used
- template/profile id + version
- render kind, language, (optional) projection
- determinism mode
- status + refusal/error code (if not `ok`)
- trace coverage metrics (assertions rendered / omitted / uncertain / refused)

---

## Links

- [Lifecycle](Lifecycle)
- [Artifacts](Artifacts) (Render Bundle)
- [Operations](Operations) (release/rollback behavior that keeps the runtime safe)
- [Components — Orgo](Components-Orgo)
- [Components — Konnaxion & Malkuth](Components-Konnaxion-and-Malkuth)