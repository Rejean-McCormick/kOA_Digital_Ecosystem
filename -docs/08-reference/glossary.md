# Glossary

## Core architecture terms

**Sephira (Node)**  
A system function cluster with concrete responsibilities and boundaries. :contentReference[oaicite:6]{index=6}

**Path**  
A typed interface boundary with a payload contract and constraints. :contentReference[oaicite:7]{index=7}

**Canonical truth**  
Truth is canonical only after validation and compilation into Kristal Exchange (content-addressed, immutable). :contentReference[oaicite:8]{index=8}

**Governed change**  
Changes produce new artifacts/versions; no in-place mutation of canonical truth. :contentReference[oaicite:9]{index=9}

**Truth boundary**  
The acceptance boundary where outputs become canonical: validation passes, then compilation produces Exchange/Pack. :contentReference[oaicite:10]{index=10}

**Deterministic gate**  
A pass/fail boundary that deterministically decides whether the pipeline may proceed (e.g., “no compile on fail”). :contentReference[oaicite:11]{index=11} :contentReference[oaicite:12]{index=12}

**Fail-closed**  
If hashes/signatures are declared and verification does not verify, the system must reject/stop rather than proceed “best-effort.” :contentReference[oaicite:13]{index=13}

**Content-addressed ID**  
An identifier derived from artifact content so identical content yields identical IDs (used for Exchange and Runtime Packs). :contentReference[oaicite:14]{index=14} :contentReference[oaicite:15]{index=15}

**Canonicalization (JCS / RFC 8785)**  
A standard method for canonical JSON serialization used to make hashes stable across toolchains. :contentReference[oaicite:16]{index=16}


## Canon pivot (Da’at) terms

**Da’at (Truth pivot)**  
The system’s canonical truth storage + offline-executable distribution form (implemented as Kristal Exchange + Runtime Pack). :contentReference[oaicite:17]{index=17} :contentReference[oaicite:18]{index=18}

**Kristal (v3)**  
Standard + compilation pipeline + runtime pack approach for validated knowledge distribution. :contentReference[oaicite:19]{index=19}

**Kristal Exchange (Exchange)**  
Canonical, content-addressed, auditable source of truth for validated knowledge; mergeable and comparable across toolchains. :contentReference[oaicite:20]{index=20}

**Kristal Runtime Pack (Runtime Pack)**  
Derived, offline-executable indexed form for constrained queries; optimized for offline distribution and predictable execution. :contentReference[oaicite:21]{index=21}

**Manifest**  
A schema-defined record describing build/provenance/policies/files for an artifact (e.g., Runtime Pack Manifest). :contentReference[oaicite:22]{index=22}


## Pipeline representation terms

**Ingest**  
Stage where documents/web/PDF/datasets/signals enter the pipeline. :contentReference[oaicite:23]{index=23}

**Claim-IR (Claim Intermediate Representation)**  
Extractor output: schema-constrained proposals (uncertainty + evidence) that are not yet canonical truth. :contentReference[oaicite:24]{index=24} :contentReference[oaicite:25]{index=25}

**Resolved Claim-IR**  
Resolution output from SenTient: explicit structured decisions with ambiguity preserved when unresolved. :contentReference[oaicite:26]{index=26} :contentReference[oaicite:27]{index=27}

**Validation report**  
Deterministic acceptance output (pass/fail + reasons) that gates compilation (“no compile on fail”). :contentReference[oaicite:28]{index=28}

**Compilation**  
Stage producing Exchange (canonical + signed + content-addressed) and Runtime Pack (deterministic, manifest-recorded policies). :contentReference[oaicite:29]{index=29}

**Render (Architect)**  
Deterministic generation after validation; must trace to claims/evidence and must not add facts. :contentReference[oaicite:30]{index=30} :contentReference[oaicite:31]{index=31}


## Node terms (mapped components)

**Keter (Mandate / Policy Root)**  
Defines purpose, ethical scope, and success criteria; immutable during runtime; system should not operate without an active mandate. :contentReference[oaicite:32]{index=32}

**Chokmah (Raw Input Sources)**  
Provides raw content/assets; preserve provenance; do not mutate sources. :contentReference[oaicite:33]{index=33}

**Binah (Blueprint / Schemas)**  
Defines structure: schemas, ontologies, policy bundles, build configs; deterministic and versioned. :contentReference[oaicite:34]{index=34}

**Chesed (Konnaxion — Social facet)**  
People/roles/collaboration/visibility/trust surface; does not mutate canonical truth. :contentReference[oaicite:35]{index=35}

**Gevurah (Orgo — Control plane)**  
Governance, constraints, workflow gating, lifecycle; stage-order enforcement; no compile on fail; audit trails; governed change only. :contentReference[oaicite:36]{index=36}

**Tiferet (SenTient — Resolution)**  
Semantic reconciliation: converts uncertain claims into explicit resolution structures; preserves ambiguity; no invention of facts; schema-valid outputs. :contentReference[oaicite:37]{index=37}

**Netzach (Architect-Strategy)**  
Planning: goal decomposition, prioritization, orchestration decisions; aligns with mandate and constraints; does not bypass truth boundary. :contentReference[oaicite:38]{index=38}

**Hod (Architect-Render)**  
Deterministic articulation into human outputs with provenance/trace; no new facts; trace coverage required; ambiguity explicit or refusal. :contentReference[oaicite:39]{index=39}

**Yesod (SwarmCraft)**  
Execution loop: workforce orchestration, agent dispatch, artifact assembly; faithful execution; dependency correctness; telemetry mandatory. :contentReference[oaicite:40]{index=40}

**Malkuth (EkoH — Trust ledger)**  
Expertise/ethics scoring, weighted influence, acceptance signals; immutable score history; privacy controls; does not mutate Exchange. :contentReference[oaicite:41]{index=41}


## Path terms (protocol boundaries)

**Conscience (Input trunk path)**  
External sources/UI → SenTient ingestion & reconciliation; preserves provenance; does not collapse ambiguity; deterministic schema outputs. :contentReference[oaicite:42]{index=42}

**Écriture (Formulation boundary)**  
Extractors → SenTient; payload is Claim-IR; must include evidence pointers + uncertainty; schema-invalid payloads rejected. :contentReference[oaicite:43]{index=43}

**Loi (Canonisation boundary)**  
SenTient → validation → Kristal Exchange; Resolved Claim-IR → validation report → Exchange compilation; no compile on fail; deterministic canonicalization; content-addressed truth. :contentReference[oaicite:44]{index=44}

**Ancrage (Interface trunk path)**  
Architect-Render + Konnaxion → users/EkoH; render bundles + distribution/UX + social acceptance surfaces; no new facts; trace_map; verify packs fail-closed; rollback-safe distribution. :contentReference[oaicite:45]{index=45}


## Artifact & contract terms

**SmartCell**  
Atomic data contract used to transport ingested content/candidate evidence into SenTient; must preserve provenance and ambiguity; schema-valid. :contentReference[oaicite:46]{index=46} :contentReference[oaicite:47]{index=47}

**Case (Orgo)**  
Long-lived container for an incident/situation/pattern/theme; aggregates tasks, labels, severity, participants, and context. :contentReference[oaicite:48]{index=48}

**Task (Orgo)**  
Canonical unit of work with a shared schema (type/category/subtype/label/status/priority/severity/visibility/ownership/deadlines/escalation/metadata). :contentReference[oaicite:49]{index=49} :contentReference[oaicite:50]{index=50}

**Label (Orgo)**  
Structured information label encoding vertical scope and information intent plus optional horizontal role: `<BASE>.<CATEGORY><SUBCATEGORY>.<HORIZONTAL_ROLE?>`. :contentReference[oaicite:51]{index=51}

**Profile (Orgo)**  
Template defining reactivity, transparency, review cadence, retention, pattern sensitivity, logging depth, and automation posture for an organization. :contentReference[oaicite:52]{index=52} :contentReference[oaicite:53]{index=53}

**Render bundle**  
Renderer output containing text + trace map + refusal codes (deterministic articulation). :contentReference[oaicite:54]{index=54}

**Trace map**  
Mapping that links rendered factual assertions back to canonical statement/claim/evidence identifiers; used to enforce “no new facts.” :contentReference[oaicite:55]{index=55}

**Refusal code**  
Deterministic error/refusal classification produced by renderers when requirements cannot be met (e.g., missing trace). :contentReference[oaicite:56]{index=56}


## Trust and voting terms (EkoH / Smart Vote)

**UserExpertiseScore**  
User’s current expertise score per domain (raw_score, weighted_score). :contentReference[oaicite:57]{index=57}

**UserEthicsScore**  
Ethical weight multiplier applied to a user’s scores. :contentReference[oaicite:58]{index=58}

**ScoreHistory**  
Audit trail of all score changes (old_value, new_value, change_reason). :contentReference[oaicite:59]{index=59}

**ConfidentialitySetting**  
Per-user privacy level for identity display alongside scores (public/pseudonym/anonymous). :contentReference[oaicite:60]{index=60}

**Vote (Smart Vote)**  
Stores each user vote with raw and weighted values; weighting uses EkoH-based influence. :contentReference[oaicite:61]{index=61}

**VoteResult**  
Aggregated voting result per target (sum_weighted_value, vote_count). :contentReference[oaicite:62]{index=62}
