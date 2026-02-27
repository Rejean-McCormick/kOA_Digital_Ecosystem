# EkoH Scores & Votes (Trust Ledger Artifacts)

## Summary
EkoH is the ecosystem’s trust/impact registry: expertise and ethics scoring, weighted influence, and audit trails of change. It is a **parallel ledger**: it observes validated outcomes and community signals, and it must not mutate canonical truth (Kristal Exchange).  

This file defines the canonical *data artifacts* and *contracts* used by EkoH and Smart Vote.

---

## Scope
This specification covers:
- Expertise and ethics score artifacts
- Score history and configuration artifacts (audit + tuning)
- Privacy/confidentiality settings for displaying identity with scores
- Weighted voting artifacts (Vote, VoteResult, modalities)
- Update semantics (atomicity, immutability, auditability)

---

## Core invariants (hard)
1) **Ledger immutability**
   - Score change events are append-only. Corrections are new entries, not edits.
2) **No mutation of canon**
   - EkoH must not change Kristal Exchange (Da’at) or truth artifacts directly.
3) **Weighted fairness**
   - Raw inputs (votes, signals) are translated into weighted outcomes using expertise + ethics.
4) **Privacy controls**
   - Display identity must honor confidentiality settings.
5) **Atomic update semantics**
   - When a validated outcome triggers a score update, the outcome record and the score update must be processed atomically or in correct transactional order.
6) **Auditability**
   - A user’s current score must be derivable from ScoreHistory + initial state.
7) **No direct user editing**
   - Users cannot directly edit their score; updates come from system-recorded events and governed processes.

---

## Artifact catalog (canonical list)

### A) Expertise taxonomy
#### ExpertiseCategory
Catalog of expertise domains.
- `id` (PK)
- `name` (unique domain name)

---

### B) Expertise scores
#### UserExpertiseScore
Current expertise score per user per domain.
- `id` (PK)
- `user` (FK → User)
- `category` (FK → ExpertiseCategory)
- `raw_score` (numeric)
- `weighted_score` (numeric)

**Semantics**
- `raw_score` is the unweighted merit signal.
- `weighted_score` reflects any applied weights (e.g., ethics multiplier, configuration parameters).

---

### C) Ethics score
#### UserEthicsScore
Ethical weight multiplier for a user.
- `user` (OneToOne FK → User, also PK)
- `ethical_score` (numeric)

**Semantics**
- Default may be 1.0.
- Violations reduce the multiplier; positive conduct may raise it (if allowed by policy).
- This value influences weighting across the ledger (votes, influence computations).

---

### D) Score tuning and trace
#### ScoreConfiguration
Named weight parameters for score calculations.
- `id` (PK)
- `weight_name` (string; parameter name)
- `weight_value` (numeric)
- `field` (nullable; identifies which field the weight applies to)

#### ContextAnalysisLog
Log of AI-driven contextual adjustments to sub-scores.
- `id` (PK)
- `entity_type` (string)
- `entity_id` (string/int)
- `field` (string; score field adjusted)
- `input_metadata` (JSON)
- `adjustments_applied` (JSON)

**Notes**
- ContextAnalysisLog is audit material. It must not replace ScoreHistory.

---

### E) Privacy
#### ConfidentialitySetting
Per-user display preference for identity alongside scores.
- `user` (OneToOne FK → User, also PK)
- `level` (ENUM: `public` / `pseudonym` / `anonymous`)

**Semantics**
- A user may be fully visible, pseudonymous, or anonymous in leaderboards and vote displays.
- Transparency applies to *outcomes and computations*; identity display respects confidentiality.

---

### F) Audit trail
#### ScoreHistory
Append-only audit trail of all score changes.
- `id` (PK)
- `merit_score` (FK → UserExpertiseScore)
- `old_value` (numeric)
- `new_value` (numeric)
- `change_reason` (string)

**Required behavior**
- Every score change MUST generate a ScoreHistory entry.
- Manual overrides (if permitted) MUST be logged with explicit reasons.
- Deletions/edits are forbidden; corrections are new history entries.

---

### G) Weighted voting (Smart Vote)
#### Vote
Individual vote record with raw and weighted values.
- `id` (PK)
- `user` (FK → User)
- `target_type` (string identifier of what is being voted on)
- `target_id` (ID of target entity)
- `raw_value` (numeric)
- `weighted_value` (numeric)

**Semantics**
- `weighted_value` is computed using current trust weights (expertise + ethics).
- Raw votes are preserved for transparency and auditing.

#### VoteModality
Voting mode configuration.
- `id` (PK)
- `name` (unique modality name)
- `parameters` (JSON)

#### VoteResult
Aggregated vote result per target.
- `id` (PK)
- `target_type`
- `target_id`
- `sum_weighted_value` (numeric)
- `vote_count` (int)

#### EmergingExpert
Flags rapid merit growth.
- `id` (PK)
- `user` (FK)
- `detection_date` (date)
- `score_delta` (numeric)

#### IntegrationMapping
Cross-module mapping for voting contexts.
- `id` (PK)
- `module_name` (string)
- `context_type` (string)
- `mapping_details` (JSON)

---

## Computation contracts (normative)

### Weighting contract
- Any subsystem that computes influence MUST use:
  - domain expertise weights (UserExpertiseScore),
  - ethics multiplier (UserEthicsScore),
  - and ScoreConfiguration parameters consistently.
- Raw inputs remain raw in storage; weighting produces derived values for outcomes.

### Auditability contract
A verifier must be able to:
1) Fetch current score (UserExpertiseScore.weighted_score)
2) Reconstruct the score from ScoreHistory + config (within defined tolerances)
3) Validate that each change has a reason and a source event reference (recommended)

### Privacy contract
- ConfidentialitySetting controls *identity display*, not ledger correctness.
- Aggregated outcomes may be shown while identity is pseudonymous/anonymous.

---

## Update semantics (events in, updates out)

### Inputs (events)
EkoH updates are driven by structured system events such as:
- task validated as complete
- content build published
- vote cast / consultation closed
- moderation/ethics outcomes

Each event SHOULD include:
- actor (user or pseudo-user for agents)
- action type
- outcome status (validated / rejected)
- domain/category
- references (case_id/task_id/build_id or content IDs)

### Outputs
- Score updates (UserExpertiseScore / UserEthicsScore)
- Append-only ScoreHistory entries
- VoteResult aggregates (when applicable)

---

## Failure modes (recommended stable codes)
- `CONFIDENTIALITY_LEVEL_INVALID`
- `SCORE_CONFIG_MISSING`
- `SCORE_UPDATE_NOT_ATOMIC`
- `SCORE_HISTORY_WRITE_FAILED`
- `VOTE_TARGET_UNKNOWN`
- `VOTE_WEIGHT_COMPUTATION_FAILED`
- `AUDIT_RECONSTRUCTION_MISMATCH`
- `UNAUTHORIZED_SCORE_MUTATION_ATTEMPT`

---

## Conformance tests (minimum)
1) **Immutability**
   - Attempt to edit/delete ScoreHistory must be rejected.
2) **Atomicity**
   - A validated outcome that awards score must not produce “score updated without outcome recorded” (and vice versa).
3) **Weighted fairness**
   - Vote weighted_value equals raw_value transformed by declared weights.
4) **Privacy**
   - ConfidentialitySetting controls identity display modes correctly.
5) **No canon mutation**
   - EkoH update routines have no write path to Exchange/Runtime Pack artifacts.
6) **Auditability**
   - Recompute current scores from ScoreHistory and reach the stored current value.

---

## Notes / TODO
- Define a canonical “EkoH event envelope” schema used by Orgo/SwarmCraft/Publishing to trigger updates.
- Specify the weighting formula precisely (including normalization, decay, caps).
- Define governance rules for manual overrides (if any) and require explicit logging categories.
