# Kristal v4 — Contract Pointers (kOA integration)

**Normative for kOA:** YES  
**External normative reference:** Kristal v4 (pinned)

**Purpose:** Provide the canonical pointers from kOA → Kristal v4 normative contracts (schemas/spec).  
**Rule:** kOA does not redefine Kristal artifact formats. If anything here conflicts with Kristal v4, Kristal v4 wins.

See also:
- `40-integration/kristal-v4/pinned-dependency.md`
- `40-integration/kristal-v4/koa-profile.md`

---

## 1) Mandatory inter-system interface contracts (Kristal-owned)

> **Important:** All Kristal-owned references in kOA docs MUST go through the pinned dependency path (e.g., `vendor/kristal/...`).

### 1.1 Claim-IR (proposal boundary)
**Kristal source**
- `vendor/kristal/02-schemas/claim-ir.schema.json`

**kOA usage**
- Produced by extractors; consumed by SenTient and validation.

**kOA pointers**
- `10-system/lifecycle.md`
- `20-nodes/tiferet-sentient.md`

---

### 1.2 Resolved Claim-IR (resolution boundary)
**Kristal source**
- `vendor/kristal/02-schemas/resolved-claim-ir.schema.json`

**kOA usage**
- Produced by SenTient; consumed by validation and compilation.

**kOA pointers**
- `10-system/lifecycle.md`
- `20-nodes/tiferet-sentient.md`

---

### 1.3 Validation Report (acceptance gate)
**Kristal source**
- `vendor/kristal/02-schemas/validation-report.schema.json`

**kOA usage**
- Produced by validation; used by Orgo to authorize compilation. Validation failure blocks compilation.

**kOA pointers**
- `10-system/lifecycle.md`
- `20-nodes/gevurah-orgo.md`
- `20-nodes/yesod-compiler.md`
- `50-operations/pipeline.md`

---

### 1.4 Exchange commit contract (canon boundary)
**Kristal source**
- Manifest schema: `vendor/kristal/02-schemas/exchange-manifest.schema.json`
- Federation/shard schemas (if used):
  - `vendor/kristal/02-schemas/exchange-federation-manifest.schema.json`
  - `vendor/kristal/02-schemas/exchange-shard-manifest.schema.json`

**kOA usage**
- Produced by Kristal compiler; consumed by query/render/export.

**kOA pointers**
- `10-system/lifecycle.md`
- `20-nodes/daat-kristal-bridge.md`
- `20-nodes/yesod-compiler.md`

---

### 1.5 Runtime Pack contract (offline execution boundary)
**Kristal source**
- Manifest schema: `vendor/kristal/02-schemas/runtime-pack-manifest.schema.json`

**kOA usage**
- Produced by Kristal compiler; distributed/verified/activated by Konnaxion; queried offline.

**kOA pointers**
- `10-system/lifecycle.md`
- `20-nodes/chesed-konnaxion.md`
- `20-nodes/daat-kristal-bridge.md`
- `50-operations/releases.md`
- `50-operations/rollback.md`

---

## 2) Core normative rules (Kristal-owned; do not restate)

kOA implementers MUST follow Kristal v4 rules (see Kristal spec + test vectors for exact wording and fixtures), including:

- **Schema conformance:** Exchange and Runtime Pack manifests MUST conform to their Kristal schemas.
- **No compile on fail:** validation failure blocks compilation.
- **Rebuild determinism:** identical inputs/compiler/config/policies reproduce identical IDs and declared file hashes.
- **Signature envelope:** signatures are separated so hashed content is unambiguous.
- **Fail-closed integrity:** declared hashes/signatures/key refs must verify or the consumer fails closed.
- **Forward compatibility:** unknown non-integrity fields may be ignored; integrity fields are never best-effort.

---

## 3) Query contract and profiles (Kristal-owned)

**Core query contract**
- `vendor/kristal/04-query/query-contract.md`

**Optional profiles (examples)**
- `vendor/kristal/05-profiles/profile-query-tpf-pagination.md`
- `vendor/kristal/05-profiles/profile-jsonld-export.md`
- `vendor/kristal/05-profiles/profile-rdf-wdqs-export.md`
- `vendor/kristal/05-profiles/profile-rdf-integrity-rdfc.md`
- `vendor/kristal/05-profiles/profile-provenance-nanopub-provo.md`

**Test vectors**
- `vendor/kristal/09-test-vectors/` (canonicalization/hash fixtures; validation pass/fail fixtures; query determinism fixtures)

---

## 4) Naming / compatibility notes (integration guidance)

- When Kristal declares `runtime_pack_id`, other integration surfaces may call it `pack_id`; treat them as synonyms unless a Kristal profile says otherwise.
- Prefer Kristal’s declared canonicalization identifiers exactly as recorded in Kristal schemas/manifests.

---

## 5) kOA policy surfaces (kOA-owned)

Anything that is a **policy choice** rather than an artifact format lives in kOA and must be recorded/pinned in build/release records, for example:
- rollout/activation policy (channels/cohorts, rollback triggers)
- key management and trust root pinning policy
- resource caps for validation, compilation, and query
- operational SLOs/alerting requirements

See:
- `40-integration/kristal-v4/koa-profile.md`
- `50-operations/`
- `30-artifacts/build-record.md`
- `30-artifacts/release-record.md`

---

## 6) Non-redundancy rule (kOA enforcement)

- Do not copy Kristal schema field lists or canonicalization rules into kOA docs.
- Do not maintain parallel “kOA versions” of Kristal contracts.
- Reference the pinned Kristal dependency (`vendor/kristal/...`) and link to the exact schema/spec path.