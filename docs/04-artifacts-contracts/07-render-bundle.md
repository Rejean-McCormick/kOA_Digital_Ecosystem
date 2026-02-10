# Render Bundle (Contract)

A Render Bundle is the canonical **output artifact** of **Architect-Render**. It packages:
- the **final rendered output(s)** (text/UI/structured),
- a **trace_map** that proves every factual assertion is grounded in upstream validated artifacts,
- and **render metadata** needed for reproducibility and audit.

Render Bundles are **derived artifacts**. They MUST be traceable to canon (Kristal Exchange) and/or a pinned Runtime Pack, plus pinned templates and parameters.

This document is the normative contract description aligned to `schemas/render-bundle.schema.json`. The schema is authoritative for field names and required/optional fields.

---

## 1) Purpose

Render Bundles exist to:
- produce user-facing deliverables **without introducing new facts**
- guarantee **traceability** from output back to canon and source evidence
- support **deterministic reproduction** of outputs
- provide a clean boundary between **truth (canon)** and **articulation (render)**

---

## 2) Ownership and lifecycle position

- **Producer**: Architect-Render (Hod)
- **Consumers**:
  - Konnaxion (serving/publishing outputs)
  - Humans (review, audit)
  - Orgo (optional: governance, QA)
- **Inputs (pinned)**:
  - `inputs.exchange_ref` and/or `inputs.runtime_pack_ref`
  - `inputs.blueprint_hash`
  - `inputs.policy_id`
  - `inputs.pipeline_refs` (at minimum: Resolved Claim-IR + Validation Report)
  - optional: template identity (`template.*`) and render parameters (`parameters`)

---

## 3) Normative constraints (MUST / MUST NOT)

### 3.1 No-new-facts rule
- Render outputs **MUST NOT introduce any factual statement** that is not supported by upstream validated artifacts.
- If support is missing, Architect-Render MUST do one of (policy-defined, deterministic):
  1) omit the statement,
  2) mark it explicitly as **uncertain** (only if upstream encodes uncertainty/ambiguity),
  3) **refuse** the render with a deterministic refusal code (strict policies).

### 3.2 Trace coverage
- Every factual assertion in rendered outputs MUST be represented as a trace entry with:
  - `kind = factual`
  - `state = TRACED`
  - `supports[]` non-empty
- Supports MUST resolve to validated lineage objects:
  - Exchange record/statement OR Resolved Claim-IR claim OR (if allowed) evidence pointer,
  - with references that can be traced back through: Input Snapshot → Claim-IR → Resolved Claim-IR → Validation Report → Exchange/Pack.

### 3.3 Determinism
Given identical:
- `inputs.exchange_ref` / `inputs.runtime_pack_ref`,
- `inputs.blueprint_hash`, `inputs.policy_id`, `inputs.pipeline_refs`,
- `template` (if used),
- `parameters` (if they affect output),
- renderer identity/version and determinism/config declarations,

…the Render Bundle outputs (including trace ordering) MUST be identical in deterministic mode (byte-identical or canonical-identical under the declared canonicalization profile).

### 3.4 Explicit ambiguity
If upstream encodes ambiguity:
- output MUST preserve it explicitly (alternatives, ranges, “unknown/contested”),
- trace entries MUST reflect this with `kind=uncertain` (or policy-approved representation) and supports pointing to upstream ambiguity/unresolved structures.

---

## 4) Data model

A Render Bundle is an envelope containing:
- identity + producer (renderer) metadata,
- pinned inputs and lineage references,
- one or more output artifacts,
- a trace map (assertion-level grounding),
- deterministic enforcement events (omissions/refusals/uncertainties),
- optional integrity envelope (hash/signatures).

### 4.1 Top-level envelope (minimum aligned to schema)

```json
{
  "bundle_version": "1.0",
  "bundle_id": "rb_sha256_...",

  "created_at": "2026-02-09T00:00:00Z",

  "renderer": {
    "component": "Architect-Render",
    "version": "x.y.z",
    "instance_id": "render-01",
    "build": "build_123"
  },

  "determinism": {
    "mode": "strict",
    "canonicalization_profile": "jcs-rfc8785@1",
    "stable_ordering": true,
    "config_hash": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "seed_policy": "forbidden"
  },

  "template": {
    "template_id": "tpl_...",
    "template_version": "vY",
    "locale": "en"
  },

  "parameters": {
    "audience": "internal",
    "tone": "neutral"
  },

  "inputs": {
    "exchange_ref": {
      "ref_type": "content_addressed",
      "ref": "kristal:exchange:sha256:...",
      "hash": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "hash_algorithm": "sha256",
      "byte_size": 1234
    },

    "blueprint_hash": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
    "policy_id": "policy.core.strict@1",

    "pipeline_refs": {
      "resolved_claim_ir": {
        "ref_type": "content_addressed",
        "ref": "sha256:...",
        "hash": "cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc",
        "hash_algorithm": "sha256"
      },
      "validation_report": {
        "ref_type": "content_addressed",
        "ref": "sha256:...",
        "hash": "dddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddd",
        "hash_algorithm": "sha256"
      }
    }
  },

  "outputs": [
    {
      "artifact_id": "out_1",
      "media_type": "text/markdown",
      "language": "en",
      "title": "Report",
      "content": "..."
    }
  ],

  "trace_map": {
    "version": "1.0",
    "coverage_policy": "all_factual_must_trace",
    "coverage": {
      "assertions_total": 1,
      "assertions_traced": 1,
      "assertions_omitted": 0,
      "assertions_uncertain": 0,
      "assertions_refused": 0
    },
    "entries": [
      {
        "assertion_id": "a_0001",
        "artifact_id": "out_1",
        "assertion_text": "X is a Y.",
        "kind": "factual",
        "state": "TRACED",
        "supports": [
          {
            "source_kind": "exchange_record",
            "exchange": {
              "exchange_ref": {
                "ref_type": "content_addressed",
                "ref": "kristal:exchange:sha256:...",
                "hash": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                "hash_algorithm": "sha256"
              },
              "record_id": "stmt_123"
            }
          }
        ],
        "output_span": { "start_char": 0, "end_char": 9 }
      }
    ]
  }
}
````

---

## 5) Outputs

An output is one render target (document, section, UI payload, data export).

Schema-aligned output object (RenderedArtifact):

```json
{
  "artifact_id": "out_1",
  "media_type": "text/markdown",
  "language": "en",
  "title": "Report",
  "content": "..."
}
```

Rules:

* If multiple outputs exist, each MUST have a unique `artifact_id`.
* Each trace entry MUST reference the output it belongs to via `artifact_id`.
* Output content MUST be provided either as `content` or as `content_ref` (exclusive).

---

## 6) trace_map (grounding contract)

The `trace_map` is the machine-verifiable grounding layer.

### 6.1 Trace entry model (minimum aligned to schema)

Each entry binds an output assertion to support pointers and an enforcement state:

```json
{
  "assertion_id": "a_0001",
  "artifact_id": "out_1",
  "assertion_text": "X is a Y.",
  "kind": "factual",
  "state": "TRACED",
  "supports": [
    {
      "source_kind": "resolved_claim",
      "resolved_claim": {
        "resolved_claim_ref": { "ref_type": "content_addressed", "ref": "sha256:...", "hash": "...", "hash_algorithm": "sha256" },
        "claim_id": "c_001"
      }
    }
  ],
  "output_span": { "start_char": 120, "end_char": 178 }
}
```

### 6.2 Support requirements

* If `kind=factual` AND `state=TRACED`: `supports[]` MUST be non-empty.
* If `state=UNCERTAIN`: supports MUST point to upstream ambiguity/unresolved structures (or explicitly uncertain resolved claims).
* If `state=OMITTED` or `state=REFUSED`: supports MAY be absent; the reason MUST be recorded in `events` (omission/refusal objects) with a deterministic code.

### 6.3 Span model

* For text outputs, spans SHOULD use character offsets (`start_char`, `end_char`) and MAY include line offsets.
* For structured/UI payloads, spans MAY use anchors (`anchor_start`, `anchor_end`) or other policy-approved selectors. The span style MUST be consistent per output type.

---

## 7) Enforcement events: omissions, refusals, uncertainties

Render Bundles MAY include an `events` object for deterministic enforcement artifacts:

* `events.omissions[]`: statements omitted due to trace gaps or policy rules
* `events.uncertainties[]`: statements marked uncertain due to upstream ambiguity/unresolved states
* `events.refusals[]`: full or partial refusal events (strict policy)

### 7.1 Refusal behavior

Strict policies SHOULD refuse when:

* any factual assertion cannot be trace-covered,
* required ambiguity handling is missing,
* required inputs are not pinned (missing exchange/pack refs or missing pipeline refs),
* determinism policy is violated.

Refusal encoding:

* add `events.refusals[]` entries with deterministic `code`
* set relevant trace entries to `state=REFUSED` (if entries were produced) and update `trace_map.coverage`.

### 7.2 Partial output behavior

If policy permits partial output:

* untraceable factual assertions MUST be omitted or marked uncertain (only when upstream supports uncertainty),
* `events.omissions[]` and/or `events.uncertainties[]` MUST document the action,
* `trace_map.coverage` MUST reflect the final counts.

---

## 8) Serialization and ordering

To support determinism:

* `outputs[]` SHOULD be ordered deterministically (e.g., lexicographic by `artifact_id`).
* `trace_map.entries[]` SHOULD be ordered deterministically (e.g., lexicographic by `assertion_id`).
* Within a trace entry, `supports[]` SHOULD be ordered deterministically (e.g., exchange record first, then resolved claims, then evidence pointers; stable tie-breakers within each group).

---

## 9) Security / privacy hooks

If confidentiality rules exist:

* outputs MAY include redaction markers inside content, or policy-approved `extensions` metadata.
* trace_map MUST NOT leak protected data; it may use opaque references where necessary.
* If outputs are public, systems MAY produce:

  * a public Render Bundle (safe trace pointers),
  * and a private Render Bundle (full provenance) gated by permissions.

---

## 10) Conformance tests (required)

A compliant Architect-Render MUST pass:

* **No-new-facts**: factual assertions are trace-covered or omitted/uncertain/refused per policy
* **Trace completeness**: trace entries map to real output spans/anchors
* **Determinism**: same inputs/template/params → identical bundle in deterministic mode
* **Ambiguity correctness**: upstream ambiguity becomes explicit uncertainty/alternatives with correct supports
* **Refusal determinism**: same trace gaps → same codes and same affected assertion IDs/spans
* **Schema compliance**: required fields and enums are valid (schema is authoritative)


