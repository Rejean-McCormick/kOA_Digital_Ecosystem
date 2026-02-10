# Anchoring / Interface Path (Ancrage)

**File:** `docs/05-protocols-flows/03-anchoring-interface-path.md`  
**Status:** Canonical (normative)  
**Purpose:** Specify the “last mile” contract: how validated knowledge is delivered to humans and how human response is captured—without breaking truth invariants.

---

## 1) Definition

**Anchoring (Ancrage)** is the interface trunk connecting:
- **Producer:** Architect-Render + Konnaxion  
- **Consumer:** end users / orgs + EkoH (trust/impact ledger) :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

This path exists to convert technical success (compiled truth + packs) into **human comprehension, adoption, and governed improvement**, while preserving deterministic rigor. :contentReference[oaicite:2]{index=2}

---

## 2) Scope

Anchoring covers:
1) **Distribution safety**: make the right Runtime Pack active on the right channel/device safely. :contentReference[oaicite:3]{index=3}  
2) **Deterministic articulation**: render human-facing outputs (text/cards/UI) with trace coverage and refusal semantics. :contentReference[oaicite:4]{index=4}  
3) **User interaction capture**: collect structured feedback/telemetry and route it back as governed change (Cases/Tasks), not canon mutation. :contentReference[oaicite:5]{index=5} :contentReference[oaicite:6]{index=6}  

Non-goal: anchoring is not a place where truth changes; it is a place where **truth is consumed** and **signals are produced**.

---

## 3) Producers, consumers, and payloads

### 3.1 Producer → Consumer (interface payload)
**Architect-Render + Konnaxion → Users / orgs / EkoH** :contentReference[oaicite:7]{index=7}

**Payload MUST include:**
- **Render Bundle** (rendered output + trace_map + render_metadata). :contentReference[oaicite:8]{index=8}  
- **UI surfaces** that expose the output and allow structured response capture (votes, comments, corrections, usage). :contentReference[oaicite:9]{index=9}  

### 3.2 Distribution sub-boundary (incarnation payload)
**Kristal/Orgo → Konnaxion distribution facet**  
**Payload:** Runtime Pack manifest + pack payload + channel/activation metadata. :contentReference[oaicite:10]{index=10}

---

## 4) Non-negotiable constraints (normative)

### 4.1 No new facts (hard constraint)
Architect MUST NOT introduce any factual assertion not supported by validated inputs. :contentReference[oaicite:11]{index=11}

### 4.2 Trace coverage (mandatory)
Every factual statement in the rendered output MUST have at least one support pointer in `trace_map`. If it cannot be supported, Architect MUST omit it, render it explicitly as uncertainty (only if the uncertainty exists in input), or fail deterministically. :contentReference[oaicite:12]{index=12}

### 4.3 Ambiguity preservation
If input contains unresolved ambiguity, Architect MUST render ambiguity explicitly or refuse the claim as fact; it MUST NOT silently pick a disambiguation. :contentReference[oaicite:13]{index=13}

### 4.4 Fail-closed distribution and activation
If a pack declares hashes/signatures/kid, Konnaxion MUST verify before activation and MUST fail closed on any verification failure. :contentReference[oaicite:14]{index=14}

Trust roots MUST be pinned per channel and verifiable offline (must not depend on network fetch at activation time). :contentReference[oaicite:15]{index=15}

### 4.5 Atomic activation + deterministic rollback
Activation MUST be atomic (no partial activation). :contentReference[oaicite:16]{index=16}  
Rollback MUST be deterministic given the same trigger event sequence, and Konnaxion MUST support pinned rollback and/or last-known-good rollback. :contentReference[oaicite:17]{index=17}

### 4.6 Feedback is governed and non-mutating
User feedback and telemetry MUST NOT mutate Exchange directly; they must create governed work (Cases/Tasks) or distribution adjustments. :contentReference[oaicite:18]{index=18} :contentReference[oaicite:19]{index=19}

---

## 5) Interface UX requirements (normative where stated)

### 5.1 “Why this?” trace exposure (recommended → normative if implemented)
The interface SHOULD provide “Why this recommendation?” / “Show sources” behaviors that reveal trace-backed support, so outputs are not a black box. :contentReference[oaicite:20]{index=20}

### 5.2 Accessibility
Anchoring SHOULD ensure accessibility (multi-language support and inclusive UI practices), without altering factual content across locales. :contentReference[oaicite:21]{index=21} :contentReference[oaicite:22]{index=22}

### 5.3 Feedback loop integrity (mandatory)
Votes/comments/corrections must be captured in a structured, non-tampered way (subject to moderation policy), and outcomes may be weighted by EkoH. :contentReference[oaicite:23]{index=23} :contentReference[oaicite:24]{index=24}

### 5.4 Non-disruption (deployment guidance)
Anchoring SHOULD prefer gradual rollouts (blue/green, cohorts) to reduce user shock and preserve trust; this is a deployment strategy aligned with release/activation gating. :contentReference[oaicite:25]{index=25} :contentReference[oaicite:26]{index=26}

---

## 6) Telemetry and signals (non-mutating)

Anchoring telemetry focuses on:
- **engagement/acceptance** (view time, clicks, adoption), :contentReference[oaicite:27]{index=27}
- **feedback metrics** (votes, suggestions, comments sentiment where permitted), :contentReference[oaicite:28]{index=28}
- **trust metrics** (EkoH evolution and weighted outcomes), :contentReference[oaicite:29]{index=29}
- **runtime health** (query errors, verification failures, activation failures). :contentReference[oaicite:30]{index=30} :contentReference[oaicite:31]{index=31}

Telemetry MUST:
- include correlation identifiers linking output → pack_id/kristal_id → build/release, when available, :contentReference[oaicite:32]{index=32}
- avoid embedding new facts into canon; it is operational signal only. :contentReference[oaicite:33]{index=33}

---

## 7) Failure modes (deterministic)

### 7.1 Rendering failures (Architect-Render)
- Missing trace coverage → deterministic refusal/error. :contentReference[oaicite:34]{index=34}  
- Ambiguity not renderable under constraints → refusal with reason. :contentReference[oaicite:35]{index=35}  
- Unsupported render_kind/template/profile → deterministic refusal/error. :contentReference[oaicite:36]{index=36}  

### 7.2 Distribution failures (Konnaxion)
- Signature/hash mismatch → fail closed (no activation). :contentReference[oaicite:37]{index=37}  
- Incompatible query_contract_version/policies → reject deterministically. :contentReference[oaicite:38]{index=38}  
- Revoked pack / downgrade attempt → block unless explicit rollback policy permits. :contentReference[oaicite:39]{index=39}  

### 7.3 Feedback capture failures
- Unverifiable / tampered feedback payload → reject (do not apply) and open governed review. :contentReference[oaicite:40]{index=40} :contentReference[oaicite:41]{index=41}  

---

## 8) Conformance tests (required)

A conformant Anchoring Path MUST include tests for:

1) **No-new-facts**: inject unsupported fact → must refuse or omit. :contentReference[oaicite:42]{index=42}  
2) **Trace coverage**: remove support pointers → must refuse/error deterministically. :contentReference[oaicite:43]{index=43}  
3) **Ambiguity behavior**: unresolved ambiguity → must be explicit or refused. :contentReference[oaicite:44]{index=44}  
4) **Fail-closed verification**: tamper pack payload → activation blocked. :contentReference[oaicite:45]{index=45}  
5) **Atomic activation**: simulate mid-activation failure → no partial state; old pack remains active. :contentReference[oaicite:46]{index=46}  
6) **Deterministic rollback**: same trigger sequence → same rollback result. :contentReference[oaicite:47]{index=47}  
7) **Governed feedback**: feedback creates Case/Task; Exchange remains unchanged. :contentReference[oaicite:48]{index=48} :contentReference[oaicite:49]{index=49}
