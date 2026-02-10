# ADR-0010: EkoH Trust Ledger and Voting Model

## Status
Accepted (normative)

## Context
The ecosystem requires a **societal anchoring layer** that records competence, ethics, and community impact without mutating canonical truth (Da’at / Kristal). EkoH is defined as the “trust/impact registry” whose inputs are validated actions and community signals, and whose outputs are score updates, voting weights, and auditable change trails. :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

EkoH is not a control-plane component: it does not determine project flow; it is **passively updated by system outputs** and **actively queried for trust decisions**, without pushing decisions or altering core logic/data flows. :contentReference[oaicite:2]{index=2}

A weighted voting mechanism (“Smart Vote”) is required so that collective signals (votes/consultations) are translated into outcomes using **expertise + ethics weighting**, while preserving auditability and privacy controls. :contentReference[oaicite:3]{index=3} :contentReference[oaicite:4]{index=4}

## Decision
We adopt EkoH as a **parallel ledger domain** implemented in Konnaxion, with a **typed, auditable scoring model** and a **weighted voting model**.

### D1 — Core trust primitives (scores + audit)
EkoH SHALL expose the following canonical data artifacts (tables/models):
- Expertise domains: `ExpertiseCategory` :contentReference[oaicite:5]{index=5}
- Per-user per-domain expertise: `UserExpertiseScore` (raw_score, weighted_score) :contentReference[oaicite:6]{index=6}
- Per-user ethics multiplier: `UserEthicsScore` :contentReference[oaicite:7]{index=7}
- Weight parameters: `ScoreConfiguration` :contentReference[oaicite:8]{index=8}
- Contextual adjustment audit (optional): `ContextAnalysisLog` :contentReference[oaicite:9]{index=9}
- Privacy controls: `ConfidentialitySetting` (public/pseudonym/anonymous) :contentReference[oaicite:10]{index=10}
- Append-only audit trail: `ScoreHistory` :contentReference[oaicite:11]{index=11}

### D2 — Append-only ledger semantics
- Score changes MUST be append-only via `ScoreHistory` (no destructive edits); corrections are new entries. :contentReference[oaicite:12]{index=12}
- “No direct user editing” is enforced by architecture: score updates are driven by system events/outcomes and governed processes (not ad-hoc UI writes). :contentReference[oaicite:13]{index=13}

### D3 — Weighted voting (“Smart Vote”)
Smart Vote SHALL be implemented with:
- `Vote` storing raw and weighted values :contentReference[oaicite:14]{index=14}
- `VoteModality` defining voting modes (approval/ranking/rating/preferential) :contentReference[oaicite:15]{index=15} :contentReference[oaicite:16]{index=16}
- `VoteResult` as the aggregated weighted outcome per target :contentReference[oaicite:17]{index=17}
- `EmergingExpert` detection tracking rapid merit growth :contentReference[oaicite:18]{index=18}
- `IntegrationMapping` to connect vote contexts across modules :contentReference[oaicite:19]{index=19}

### D4 — Weighting rule (normative)
Raw inputs (votes and similar signals) MUST be translated into weighted outcomes using current expertise and ethics weights, configured via `ScoreConfiguration`, applied consistently. :contentReference[oaicite:20]{index=20}

### D5 — Privacy rule (normative)
Confidentiality settings MUST control **identity display** (public/pseudonym/anonymous) while maintaining transparency of outcomes and auditability of computations. :contentReference[oaicite:21]{index=21} :contentReference[oaicite:22]{index=22}

### D6 — Atomicity rule (normative)
When a validated system outcome triggers an EkoH update, the action and the corresponding ledger update MUST be processed atomically or in correct transactional order (no “score updated without validated outcome,” and no “validated outcome without ledger reflection,” when policy requires). :contentReference[oaicite:23]{index=23}

### D7 — Separation from canonical truth (normative)
EkoH MUST NOT mutate Da’at (Kristal Exchange / Runtime Pack). It is a parallel ledger that observes validated outputs and provides trust signals to consumers (UI, moderation, planning lookups). :contentReference[oaicite:24]{index=24} :contentReference[oaicite:25]{index=25}

## Rationale
- The ecosystem’s correctness depends on a strict truth boundary; trust signals must not rewrite canonical truth artifacts. :contentReference[oaicite:26]{index=26}
- Append-only score history provides transparent auditability and supports reconstructing “how a score happened,” not just its current value. :contentReference[oaicite:27]{index=27}
- Weighted voting ensures collective influence reflects demonstrated expertise and ethical trust, reducing susceptibility to low-signal participation and manipulation. :contentReference[oaicite:28]{index=28}
- Privacy controls are necessary so identity exposure can be configured without breaking ledger integrity. :contentReference[oaicite:29]{index=29}

## Consequences

### Positive
- **Auditable trust**: score outcomes can be explained via ScoreHistory and configuration.
- **Fairer collective decisions**: voting outcomes incorporate expertise/ethics weighting. :contentReference[oaicite:30]{index=30}
- **Cross-module coherence**: IntegrationMapping supports consistent trust/vote semantics across domains. :contentReference[oaicite:31]{index=31}
- **Operational alignment**: EkoH remains a passive ledger and lookup source; it does not become a hidden control plane. :contentReference[oaicite:32]{index=32}

### Costs / Tradeoffs
- Complexity: weight configuration, ethics multipliers, and modalities add governance/ops overhead.
- Requires robust event hygiene: atomic update semantics depend on reliable “validated outcome” events. :contentReference[oaicite:33]{index=33}
- Privacy adds product complexity: separate correctness (ledger) from identity display controls. :contentReference[oaicite:34]{index=34}

## Alternatives considered (rejected)

### A1 — Unweighted “one user = one vote”
Rejected because it ignores competence/ethics and weakens the “distributed trust” intent of EkoH’s role. :contentReference[oaicite:35]{index=35}

### A2 — Editing current scores in place (no history)
Rejected because it breaks auditability and allows opaque changes; ScoreHistory append-only is required. :contentReference[oaicite:36]{index=36}

### A3 — EkoH drives workflows directly (push-based governance)
Rejected because EkoH is defined as an observer/lookup system, not a controller; workflow gating belongs to Orgo. :contentReference[oaicite:37]{index=37} :contentReference[oaicite:38]{index=38}

## Implementation notes

### I1 — Configuration and thresholds (settings)
Initial coefficients and Smart Vote parameters are managed as explicit configuration (examples include vote modality choices and emerging expert thresholds). :contentReference[oaicite:39]{index=39}

### I2 — Update triggers
EkoH updates are driven by structured events emitted when actions are validated or content is published (examples: “who did what, with what result”). :contentReference[oaicite:40]{index=40}

### I3 — Consumer usage
Consumers (e.g., Architect for collaborator selection, moderation tools, frontends) query EkoH as an authorized lookup; EkoH does not push decisions. :contentReference[oaicite:41]{index=41}

## References
- Canonical EkoH + Smart Vote table/model list and fields. :contentReference[oaicite:42]{index=42}
- EkoH as Malkuth societal ledger; inputs/outputs/invariants. :contentReference[oaicite:43]{index=43}
- “Passive update / active query” behavior and “does not alter core flows.” :contentReference[oaicite:44]{index=44}
- Weighted fairness, privacy, and atomicity invariants (design intent). :contentReference[oaicite:45]{index=45}
