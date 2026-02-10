# 10 — Malkuth: EkoH (Trust & Impact Ledger)

## Purpose
**EkoH** is the ecosystem’s **societal ledger**: it records **expertise**, **ethics**, and **community impact** as an auditable, privacy-aware scoring system. EkoH is where validated outcomes become **reputational capital** and where community signals become **weighted influence** (e.g., voting weights, consultation weights, quality signals).

EkoH is not a truth engine. It **does not mutate canonical truth** (Kristal Exchange) and does not bypass governance. It is a **parallel ledger**: passively updated by validated system events and actively queried by authorized components.

---

## Responsibilities

EkoH MUST:
- Maintain per-user expertise scores by domain (merit/reputation).
- Maintain per-user ethics scores (ethical multiplier / trustworthiness).
- Maintain an immutable audit trail of score changes (append-only history).
- Provide weighting services for influence mechanisms:
  - weighted voting (Smart Vote)
  - weighted consultations / consensus
  - optional ranking and selection weights (e.g., collaborator selection)
- Enforce privacy controls for identity display (public / pseudonym / anonymous) while preserving auditability.
- Provide query APIs for:
  - user profiles (scores + summaries)
  - moderation / anomaly detection
  - authorized selection logic (e.g., Architect choosing a contributor)
  - public or semi-public transparency views (policy-dependent)

EkoH MUST NOT:
- Directly edit or alter Kristal Exchange, Runtime Packs, or any canonical truth artifact.
- Accept direct user edits to scores (no self-inflation). Updates must be event-driven and logged.
- Determine workflow gating (that is Orgo’s role). EkoH can only be used as a signal input to governed decisions.
- Remove or rewrite history entries. Corrections must be appended as new entries.

---

## Contract boundary (high-level)

### Upstream → EkoH (event ingestion)
EkoH ingests **structured events** from validated system outcomes and community interactions, such as:
- Task validated as complete (Orgo completion event)
- Content published / released (Kristal/Konnaxion release event)
- Peer feedback / votes / consultation participation (Konnaxion/EthiKos/other modules)

**Rule:** Only **validated actions** may change merit/expertise scores. Unvalidated drafts do not change scores.

### EkoH → Downstream (read/weight services)
EkoH provides:
- Current weights (expertise × ethics) for voting and influence computations
- Ledger summaries for profiles and transparency dashboards
- Audit views for administrators/auditors (respecting confidentiality policies)
- Optional “health signals” for governance (flagging anomalies, ethics drops)

EkoH is:
- **passively updated** by system outputs
- **actively queried** for trust decisions
- **not a push-based controller** of core data/logic flows

---

## Data model (canonical ledger artifacts)

EkoH is typically implemented as database-backed artifacts. The following tables/models represent the canonical minimum set.

### Expertise and ethics
- **ExpertiseCategory**
  - knowledge domains used to classify expertise
- **UserExpertiseScore**
  - current score per user per domain
  - fields (representative): `user`, `category`, `raw_score`, `weighted_score`
- **UserEthicsScore**
  - ethics multiplier per user (often one row per user)
  - fields: `user`, `ethical_score`

### Configuration and analysis
- **ScoreConfiguration**
  - named weight parameters for score calculations (global or field-specific)
  - fields: `weight_name`, `weight_value`, `field?`
- **ContextAnalysisLog**
  - AI/logic adjustments applied to sub-scores in context
  - fields: `entity_type`, `entity_id`, `field`, `input_metadata (json)`, `adjustments_applied (json)`

### Privacy
- **ConfidentialitySetting**
  - user preference for identity exposure: `public | pseudonym | anonymous`

### History (audit)
- **ScoreHistory**
  - immutable trail of all score changes
  - fields: `merit_score (FK to UserExpertiseScore)`, `old_value`, `new_value`, `change_reason`

### Weighted voting (Smart Vote)
- **Vote**
  - individual votes with raw and weighted values
  - fields: `user`, `target_type`, `target_id`, `raw_value`, `weighted_value`
- **VoteModality**
  - voting mode parameters (approval, ranking, rating, preferential)
  - fields: `name`, `parameters (json)`
- **VoteResult**
  - aggregated weighted outcomes per target
  - fields: `target_type`, `target_id`, `sum_weighted_value`, `vote_count`
- **EmergingExpert**
  - detection of fast-rising merit (optional but supported)
- **IntegrationMapping**
  - maps voting contexts across modules (cross-module weighting support)

---

## Scoring and weighting rules (normative)

### 1) Weight definition
At query time, EkoH MUST provide a **current trust weight** for a user in a given context:
- `expertise_weight(domain)` derived from `UserExpertiseScore.weighted_score`
- `ethics_multiplier` derived from `UserEthicsScore.ethical_score`
- final weight uses ScoreConfiguration rules:
  - e.g., `final_weight = f(expertise_weight, ethics_multiplier, context_weights)`

The exact function `f()` is policy-defined but MUST be:
- deterministic for a given policy/config version
- logged (policy version and config hash referenced by computations where applicable)

### 2) Voting transformation (Smart Vote)
For any vote record:
- EkoH MUST store both:
  - `raw_value` (what the user submitted)
  - `weighted_value` (result after applying trust weight)

**Invariant:** raw input (vote) is always translated into a weighted outcome using current scores under the declared policy.

### 3) Expertise updates
Expertise score updates MUST be driven by **validated events**:
- contribution accepted
- task validated as complete
- published output credited

If manual overrides exist, they MUST:
- be rare
- be clearly logged (ScoreHistory with explicit reason and actor)
- preserve auditability (no hidden adjustments)

### 4) Ethics updates
Ethics score updates may be driven by:
- violations confirmed by governance/moderation
- positive trust events (mentorship, constructive participation)
- anomaly detection outcomes (policy-defined)

Ethics changes MUST be logged and traceable.

---

## Event ingestion (recommended contract)

EkoH SHOULD be updated via an append-only event stream.

### LedgerEvent (recommended fields)
- `event_id` (unique)
- `event_type` (e.g., `TASK_VALIDATED`, `CONTENT_PUBLISHED`, `VOTE_CAST`, `ETHICS_FLAG_CONFIRMED`)
- `subject` (user or pseudo-user)
- `domain/category` (optional)
- `delta` (numeric change, if applicable)
- `references` (links to Orgo case/task IDs, Kristal IDs, pack IDs, moderation records)
- `timestamp`
- `applied_by` (system component identity)
- `policy_id` / `config_hash`

### Atomicity requirement
When an upstream validated action triggers an EkoH update:
- the completion/publication event and the ledger update MUST be applied atomically or in a verified order such that audits cannot observe “score changed without the validated action”.

---

## Privacy model (normative)

EkoH MUST support identity exposure levels per user:
- `public`: identity displayed with scores
- `pseudonym`: alias displayed; real identity hidden in public views
- `anonymous`: no identity displayed publicly; internal audit still possible under authorization

Privacy settings MUST be honored in all UI surfaces and public exports while preserving:
- integrity of weighting logic
- auditability for authorized reviewers

---

## Observability (recommended)

EkoH SHOULD emit:
- score update events with correlation IDs to upstream actions
- counters: updates applied, votes processed, anomalies flagged
- histograms: update latency, query latency
- audit consistency checks:
  - reconstructability: ScoreHistory fold equals UserExpertiseScore
  - referential integrity: every history record references a valid score entity
- privacy compliance checks: no identity leaks in pseudonym/anonymous views

---

## Failure modes and handling

### Upstream inconsistencies
- Missing references to validated actions
- Duplicate events (replay)
- Out-of-order event delivery

Handling:
- idempotent application keyed by `event_id`
- reject or quarantine events missing required references
- record all rejections with deterministic codes

### Abuse / manipulation attempts
- brigading, spam votes, coordinated influence

Handling (policy-defined):
- anomaly detection + flagging
- temporary weight damping for flagged contexts
- governance escalation (Orgo/moderation workflows)
- all mitigations logged and reviewable

### Manual intervention
- emergency corrections

Handling:
- append-only corrections
- explicit “override” codes and reasons
- reversible through compensating entries

---

## Invariants (non-negotiable)

1. **Immutable history:** ScoreHistory entries are append-only; no deletion/rewrite.
2. **No canon mutation:** EkoH cannot mutate Kristal Exchange or canonical truth artifacts.
3. **Validated-only merit:** unvalidated contributions do not change expertise.
4. **Weighted fairness:** influence computations are weighted by expertise and ethics under declared policy.
5. **Privacy-respecting transparency:** transparency of scoring is supported without violating confidentiality settings.
6. **Auditability:** current scores must be derivable from history + configuration under the same policy.
7. **Atomic outcome binding:** ledger updates must correspond to real validated outcomes, not detached events.

---

## Interfaces (implementation-agnostic)

### Core APIs (conceptual)
- `GetUserScore(user_id, domain?, policy_id?) -> {expertise, ethics, weight, privacy_level}`
- `GetUserScoreHistory(user_id, domain?, range?) -> [ScoreHistoryEntry]` (authorization-gated)
- `ComputeWeight(user_id, context) -> weight`
- `RecordVote(vote_payload) -> Vote` (stores raw and weighted)
- `GetVoteResult(target_type, target_id) -> VoteResult`
- `IngestLedgerEvent(event) -> ack` (idempotent)

Authorization and privacy enforcement are mandatory on all read surfaces.

---

## Conformance tests (minimum)

An implementation claiming EkoH conformance MUST provide tests for:
- **Append-only history:** history records cannot be updated/deleted through standard interfaces.
- **Reconstructability:** folding ScoreHistory reproduces current UserExpertiseScore.
- **Validated-only updates:** unvalidated events do not alter expertise scores.
- **Deterministic weighting:** same policy/config + same scores + same vote → same weighted_value.
- **Privacy enforcement:** confidentiality levels are respected across profile/leaderboard/export surfaces.
- **Idempotency:** event replay does not double-apply deltas; vote replay does not duplicate results.

---

## Open specification items

- Define a portable **EkoH export schema** (optional) for external auditing (e.g., signed snapshots).
- Formalize the canonical weighting function registry (policy IDs, versioning, and evaluation rules).
- Specify anomaly-detection and governance escalation as explicit, typed workflows (Orgo integration).
