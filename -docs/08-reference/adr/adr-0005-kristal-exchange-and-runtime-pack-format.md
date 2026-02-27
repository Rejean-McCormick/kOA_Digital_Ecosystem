# ADR-0005: Kristal Exchange and Runtime Pack Format

**Status:** Draft  
**Date:** 2026-02-09  
**Decision Owner:** Ecosystem Architecture  
**Scope:** Artifact formats + verification/activation rules (not deployment topology)

---

## 1) Context

The ecosystem’s truth boundary requires that “canonical truth” is only produced after deterministic validation and compilation, and that once published it is immutable (changes create new versions rather than mutating in place). This is explicitly the Da’at / truth-pivot invariant: **no compile on fail**, **Exchange immutability**, and **fail-closed verification** for distribution and activation. :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

Operationally, the system produces two “release outputs” per build: a canonical Exchange and a derived Runtime Pack. :contentReference[oaicite:2]{index=2}

We need a single, contract-driven definition of:
- what the Exchange is (and how it is identified, signed, and compared),
- what the Runtime Pack is (and how it is packaged, verified, activated, and rolled back),
- the minimum recorded metadata that makes rebuilds and offline execution reproducible.

This ADR defines that format boundary. It intentionally does **not** encode operational patterns (canaries, DLQs, circuit breakers) as schema-level objects inside Exchange or Pack formats. :contentReference[oaicite:3]{index=3}

---

## 2) Decision

### 2.1 Exchange is the canonical, content-addressed truth artifact

**Decision:** The Exchange is a canonical JSON artifact whose identity is derived from canonicalization + hashing over the Exchange payload **with signature material excluded**.

- Canonicalization is normative **RFC 8785 (JCS)**. :contentReference[oaicite:4]{index=4}  
- The hash target removes `kristal_id` and all signature/attestation fields, then hashes `JCS(target)` with SHA-256 to produce `kristal_id = "sha256:<hex>"`. :contentReference[oaicite:5]{index=5}

**Rationale:** This preserves signatures as overlays (auditor/publisher/distributor attestations) while keeping identity stable across toolchains.

---

### 2.2 Runtime Packs are derived, offline-executable distribution artifacts

**Decision:** A Runtime Pack is a packaged payload (data + indexes) paired with a required manifest that makes offline behavior and reproducibility explicit.

The manifest must declare core fields (including a content-addressed `runtime_pack_id`, an Exchange reference, the build/compiler identity, policies, and the list of files). :contentReference[oaicite:6]{index=6}

**Rationale:** Runtime Packs are intentionally *not* full SPARQL; they provide predictable offline query semantics and must record the policies that affect ordering, paging, limits, and index structures so that independent implementations can reproduce and compare outcomes. :contentReference[oaicite:7]{index=7} :contentReference[oaicite:8]{index=8}

---

### 2.3 Verification and activation are fail-closed and atomic

**Decision:** Any declared integrity material (hashes/signatures) must be enforced **fail-closed**; activation must be **atomic**; rollback must be deterministic to a last-known-good state.

- Fail-closed integrity and unambiguous signature envelopes are normative requirements. :contentReference[oaicite:9]{index=9}  
- Distribution boundary invariants require fail-closed verification plus atomic activation and deterministic rollback. :contentReference[oaicite:10]{index=10}

**Rationale:** This prevents “best-effort truth” and avoids partial activation states that would desynchronize clients.

---

## 3) Format Definition

### 3.1 Exchange on-disk representation (recommended)

An Exchange release SHOULD be distributed as:

```

exchange/
exchange.json
exchange.manifest.json
exchange.signatures.json        (optional; may also be inside manifest, depending on policy)
exports/                        (optional; profile-based)

```

**Rules:**
- `exchange.json` is the canonical knowledge payload (statements + trace/provenance pointers).
- `exchange.manifest.json` records:
  - `kristal_id`
  - canonicalization profile/version
  - the content hash of the canonicalized Exchange with signatures excluded
  - build identity and input snapshot references (minimum reproducibility surface). :contentReference[oaicite:11]{index=11} :contentReference[oaicite:12]{index=12}
- Signatures MUST be removable deterministically prior to hashing; they are overlays and must not affect `kristal_id`. :contentReference[oaicite:13]{index=13}

---

### 3.2 Signature envelope (Exchange and Pack manifests)

If present, signatures MUST:
- live in a clearly separated signature envelope,
- be verifiable offline from pinned trust roots,
- be fail-closed when required signatures fail verification. :contentReference[oaicite:14]{index=14} :contentReference[oaicite:15]{index=15}

This ADR does not decide *which* signatures are required (publisher vs auditor vs distributor). That remains a deployment policy, but the verification semantics are fixed.

---

### 3.3 Runtime Pack packaging (recommended)

A Runtime Pack SHOULD be distributed as a single archive (e.g., `.zip`/`.tar.zst`) or a directory with a stable layout:

```

runtime-pack/
runtime-pack.manifest.json
data/
triples.parquet
dictionary.bin              (optional)
indexes/
spo.index                  (optional; policy-dependent)
membership_filter.bin       (optional)
roaring_bitmaps.bin         (optional)
checksums/
files.sha256               (optional convenience, redundant if manifest lists file hashes)

```

**Rules:**
- `runtime-pack.manifest.json` is required and must include the minimum core fields and file inventory. :contentReference[oaicite:16]{index=16}
- The manifest MUST declare the policy selections that define offline semantics (ordering, paging, join limits, etc.). :contentReference[oaicite:17]{index=17}
- A pack is considered valid for activation only if:
  - the manifest verifies (schema + declared hashes/signatures),
  - the referenced file hashes match,
  - any required signature set verifies. :contentReference[oaicite:18]{index=18}

---

## 4) Pipeline Integration (contract boundary)

This ADR assumes the pipeline stages are enforced as strict gates:

Ingest → Claim-IR → Resolved Claim-IR → Validation → Exchange finalize → Runtime Pack compile → Rendering. :contentReference[oaicite:19]{index=19}

The system law is: **if validation fails, compilation MUST stop** (no Exchange update; no Pack build). :contentReference[oaicite:20]{index=20}

---

## 5) Consequences

### Positive
- Stable identity: Exchange is comparable/mergeable across toolchains because hashing is canonical and signatures are excluded from the hash target. :contentReference[oaicite:21]{index=21}
- High-assurance distribution: clients can verify offline and fail-closed before activation. :contentReference[oaicite:22]{index=22} :contentReference[oaicite:23]{index=23}
- Reproducibility: Pack behavior is explicit via recorded policies, limiting cross-implementation divergence. :contentReference[oaicite:24]{index=24}

### Costs / trade-offs
- More metadata discipline: policies and manifests must be treated as first-class outputs.
- Stronger rejection surface: fail-closed means more hard errors if key material is missing or misconfigured.
- Pack format stability pressure: once clients depend on manifest + file inventory semantics, evolution must be careful and versioned.

---

## 6) Alternatives considered (rejected)

1) **Allow signatures inside the hashed payload**  
Rejected because it creates circularity and makes identity dependent on issuer overlays, breaking cross-toolchain comparability. :contentReference[oaicite:25]{index=25}

2) **Let Runtime Packs be “implementation-defined” without a strict manifest**  
Rejected because offline determinism and portability require explicit policy declaration and a stable reproducibility surface. :contentReference[oaicite:26]{index=26}

3) **Encode operational rollout patterns inside the artifact formats**  
Rejected to keep the normative surface small and keep deployment patterns as non-normative guidance/policy outside the schemas. :contentReference[oaicite:27]{index=27}

---

## 7) References (internal)

- `01-core-spec/ids-canonicalization-hashing.md` (JCS + hash target + `kristal_id`) :contentReference[oaicite:28]{index=28}  
- `01-core-spec/signatures-trust.md` (signature envelope + fail-closed verification) :contentReference[oaicite:29]{index=29}  
- `02-schemas/exchange-manifest.schema.json` :contentReference[oaicite:30]{index=30}  
- `02-schemas/runtime-pack-manifest.schema.json` :contentReference[oaicite:31]{index=31}  
- `03-reproducibility/allowed-runtime-pack-policies.md` (portable policy surface) :contentReference[oaicite:32]{index=32}  
- Ecosystem invariants (Da’at truth pivot; distribution boundary) :contentReference[oaicite:33]{index=33} :contentReference[oaicite:34]{index=34}

