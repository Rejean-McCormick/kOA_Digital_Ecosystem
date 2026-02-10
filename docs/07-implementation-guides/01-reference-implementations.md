# Reference Implementations

**File:** `docs/07-implementation-guides/01-reference-implementations.md`  
**Status:** Informative (implementation guidance)  
**Purpose:** Provide concrete “known-good” implementation shapes that satisfy the ecosystem contracts (invariants + artifacts + deterministic gates). These are reference *instances*, not requirements.

---

## 1) How to use this document

Use these references to:
- bootstrap a minimal working system quickly,
- validate that your implementation respects the canonical contracts,
- design test harnesses and conformance suites.

Non-goal: forcing one codebase or one stack. Any conforming implementation is acceptable as long as it satisfies the contracts.

A key design stance is that concrete orchestrators are *instances* unless the ecosystem standardizes them globally. :contentReference[oaicite:0]{index=0}

---

## 2) The minimum viable reference stack (end-to-end)

A minimal conforming ecosystem can be realized as:

1) **Orgo (control plane)** orchestrates pipeline stages and enforces gating. :contentReference[oaicite:1]{index=1}  
2) **SenTient (resolution)** outputs Resolved Claim-IR (not detailed here, but positioned in the pipeline). :contentReference[oaicite:2]{index=2}  
3) **Kristal (truth pivot)** compiles:
   - **Exchange** (canonical truth),
   - **Runtime Pack** (derived offline query payload). :contentReference[oaicite:3]{index=3}  
4) **Konnaxion (distribution facet)** verifies and activates packs with rollback safety. :contentReference[oaicite:4]{index=4}  
5) **Architect-Render** renders deterministic human-facing output with trace coverage and “no new facts”. :contentReference[oaicite:5]{index=5} :contentReference[oaicite:6]{index=6}  
6) Optional **SwarmCraft** executes plans into real deliverables (files/code/assets) under strict constraints. :contentReference[oaicite:7]{index=7}

A compact conceptual summary used elsewhere is: **route → normalize → render → prove** (useful as a diagnostic lens, not a primary spec). :contentReference[oaicite:8]{index=8}

---

## 3) Reference Implementation: Orgo control plane (workflow + governance)

**Goal:** a “pipeline scheduler + policy enforcer” that guarantees ordering, gating, auditability, and reproducibility. :contentReference[oaicite:9]{index=9} :contentReference[oaicite:10]{index=10}

### 3.1 Mandatory stage ordering
Orgo MUST enforce the stage order:
**Ingest → Extract → Resolve → Validate → Compile Exchange → Compile Runtime Pack → Publish/Distribute**. :contentReference[oaicite:11]{index=11}

### 3.2 Hard gate: “no compile on fail”
If validation fails, Orgo MUST NOT invoke Exchange or Runtime Pack compilation. :contentReference[oaicite:12]{index=12}

### 3.3 Build Record as canonical operational evidence
For each build, Orgo MUST persist a Build Record containing at least:
- `build_id`, `input_snapshots[]`, `compiler.name/version`, `config_hash`,
- `canonicalization_profile/version`,
- `policy_selections`,
- outputs including `kristal_id` (Exchange) and `pack_id` + payload hashes (Runtime Pack). :contentReference[oaicite:13]{index=13}

### 3.4 Release Record model (distribution traceability)
Orgo MUST create a versioned **Release** record per publication including `kristal_id`, `pack_id`, channels/cohorts, publish timestamp, verification status. :contentReference[oaicite:14]{index=14}  
This supports the required traceability chain: **build → release → distribution**. :contentReference[oaicite:15]{index=15}

### 3.5 Integrity checks before publish (fail-closed)
Before publish/distribution, Orgo MUST verify declared hashes/signatures and MUST block publish on failure. :contentReference[oaicite:16]{index=16}

---

## 4) Reference Implementation: Kristal artifacts (Exchange + Runtime Pack)

Kristal produces two primary artifacts:  
- **Exchange:** canonical, content-addressed, auditable truth. :contentReference[oaicite:17]{index=17}  
- **Runtime Pack:** derived, offline-executable indexed form for constrained queries. :contentReference[oaicite:18]{index=18}

### 4.1 Boundaries (important)
- Writes to Kristal happen only via the formal pipeline; no component mutates Exchange arbitrarily. :contentReference[oaicite:19]{index=19}  
- Feedback does not edit Exchange in place; it becomes new work (Cases/Tasks) leading to new versions. :contentReference[oaicite:20]{index=20}

---

## 5) Reference Implementation: Konnaxion distribution facet (pack manager)

**Goal:** safe offline-first delivery/activation.

### 5.1 Activation rule (atomic + gated)
Konnaxion MUST only activate when:
1) integrity verified (if declared),
2) manifest schema-valid,
3) required files exist,
4) compatibility checks pass; and activation MUST be atomic. :contentReference[oaicite:21]{index=21}

### 5.2 Downgrade prevention + rollback determinism
Konnaxion MUST implement downgrade prevention (monotonic version and/or revocation-aware), and rollback must be explicit and deterministic. :contentReference[oaicite:22]{index=22} :contentReference[oaicite:23]{index=23}

### 5.3 Trust roots pinned (offline correctness)
Trust roots are pinned per channel and must be verifiable offline; activation must not depend on network-fetched trust roots. :contentReference[oaicite:24]{index=24}

### 5.4 Storage layout and cache policy (reference default)
A recommended storage layout supports multiple versions, atomic activation, and pinned retention. :contentReference[oaicite:25]{index=25}  
Cache policy MUST be deterministic (max disk, eviction, pinned exceptions). :contentReference[oaicite:26]{index=26}

### 5.5 Conformance test focus
Konnaxion conformance tests must cover fail-closed verification, atomic activation, downgrade prevention, rollback determinism, offline behavior, and pinned cache behavior. :contentReference[oaicite:27]{index=27}

---

## 6) Reference Implementation: Architect-Render (deterministic articulation)

**Goal:** deterministic, traceable human-facing output without inventing facts.

### 6.1 Determinism + “no new facts”
For identical validated inputs + template + language + params, Architect MUST output identical text + trace map (with explicit exclusions like timestamps if declared). :contentReference[oaicite:28]{index=28}  
Architect MUST NOT add unsupported factual assertions (“no new facts”). :contentReference[oaicite:29]{index=29}

### 6.2 Render bundle contract
Architect MUST output a **render bundle** including:
- `rendered_text`,
- `trace_map`,
- `render_metadata`. :contentReference[oaicite:30]{index=30}

### 6.3 Trace coverage (hard requirement)
Every factual statement must have support pointers in the trace map; otherwise omit, mark explicit uncertainty (only if present in input), or fail deterministically. :contentReference[oaicite:31]{index=31}

---

## 7) Reference Implementation: SwarmCraft / deterministic executor (TextCraft instance)

SwarmCraft is the execution engine turning intent into “matter” (files/artifacts). :contentReference[oaicite:32]{index=32}  
A concrete reference instance is the **TextCraft Orchestrator**: a deterministic state-machine loop:

- **SCAN → PLAN → DISPATCH → EXECUTE**, reading `matrix.json`, consulting “rules” (`story_bible/`), and writing deliverables (e.g., `manuscripts/*.md`). :contentReference[oaicite:33]{index=33} :contentReference[oaicite:34]{index=34}

This instance is valuable because it demonstrates:
- deterministic orchestration (FSM loop),
- clean separation of AI services vs core logic vs data, :contentReference[oaicite:35]{index=35}
- concrete “write-to-disk” manifestation boundary (execution produces real artifacts). :contentReference[oaicite:36]{index=36}

---

## 8) Reference Implementation: multilingual renderer subsystem (Abstract Wiki Architect instance)

A strong reference for “renderer as subsystem” is **Abstract Wiki Architect**:
- hybrid neuro-symbolic architecture combining GF-based determinism with AI, and Semantic Web grounding (Wikidata QIDs). :contentReference[oaicite:37]{index=37}  
- explicit 4-layer separation: Lexicon (data), Grammar Matrix (logic), Renderer (presentation), Context (state). :contentReference[oaicite:38]{index=38}

This is useful as an *implementation blueprint* for:
- high-coverage multilingual rendering,
- strict input/output ports (stable production path vs experimental path),
- stateful discourse planning without contaminating canonical truth. :contentReference[oaicite:39]{index=39}

---

## 9) Implementation selection guidance (practical)

Pick references based on what you’re building:

- **You need governance + auditability + releases:** start from the Orgo workflow + build/release record requirements. :contentReference[oaicite:40]{index=40} :contentReference[oaicite:41]{index=41}  
- **You need offline distribution + safety:** start from Konnaxion activation/rollback/cache rules. :contentReference[oaicite:42]{index=42} :contentReference[oaicite:43]{index=43}  
- **You need user-facing deterministic outputs:** start from Architect-Render render bundle + trace coverage. :contentReference[oaicite:44]{index=44} :contentReference[oaicite:45]{index=45}  
- **You need deterministic execution into real deliverables:** start from the TextCraft/SwarmCraft loop patterns. :contentReference[oaicite:46]{index=46}

---

## 10) What to standardize next (explicit gaps)

Some documents explicitly note missing formalization that would improve implementation interoperability:
- a fully specified **render bundle schema** (trace/refusal enums), recommended to formalize alongside Exchange/Pack specs. :contentReference[oaicite:47]{index=47}
- optional “bundle-level hashing vs profile” and revocation list representation are flagged as open questions in the Konnaxion contract set. :contentReference[oaicite:48]{index=48}
