# Truth Boundary

**Purpose:** Define where “truth” becomes canonical in the ecosystem, and enforce hard boundaries that prevent downstream components (rendering, execution, social feedback) from mutating or inventing truth.

---

## 1) Core definition

**Canonical truth** exists **only after** deterministic validation and compilation into a **Kristal Exchange** artifact (content-addressed, immutable).

Everything upstream of that point is **proposal**, not truth (even if it is “high confidence”).

---

## 2) The boundary in the pipeline

### 2.1 Pre-truth (non-canonical) artifacts

These artifacts can be created, revised, and rejected without changing canonical truth:

- **Raw inputs** (snapshots / provenance)
- **Claim-IR** (extractor output)
- **Resolved Claim-IR** (SenTient resolution output with explicit ambiguity)
- **Validation Report** (accept/reject gate output)

These artifacts MAY be iterated repeatedly; none of them is permitted to redefine canon.

### 2.2 Truth (canonical) artifacts

These artifacts are the source-of-truth surface for the rest of the ecosystem:

- **Kristal Exchange**: canonical truth (immutable, content-addressed)
- **Runtime Pack**: derived distribution payload provably derived from an Exchange (integrity-bound; activation is gated)

---

## 3) Gate rule: “No compile on fail”

Validation is a deterministic acceptance gate. If validation fails, compilation MUST NOT occur (“no compile on fail”).

This is the mechanism that makes the truth boundary enforceable rather than merely conceptual.

---

## 4) Write permissions (what is allowed to write truth)

### 4.1 Only the formal build pipeline can write Exchange
Writes that produce new canonical truth are only permitted through the formal pipeline (Orgo-controlled stages). No component may bypass gates and write canon arbitrarily.

### 4.2 No in-place mutation of canonical truth
Canonical truth is immutable once compiled into Exchange; changes MUST produce new artifacts/versions via a new governed build.

---

## 5) Downstream usage rules (what must not bypass the boundary)

### 5.1 Architect must not bypass truth
When producing user-facing outputs or operational decisions that depend on facts, Architect-Render MUST operate on validated canonical artifacts (Exchange and/or Runtime Pack plus pipeline refs). It MUST NOT treat unvalidated sources as truth.

Architect-Strategy MAY propose follow-up work using non-canonical signals, but those proposals must route through Orgo and the governed pipeline before they can affect canon.

### 5.2 Rendering: “no new facts” + trace coverage
Architect-Render is constrained by:
- **No new facts downstream**: it MUST NOT introduce factual assertions that are unsupported by validated lineage.
- **Trace coverage**: every factual assertion MUST be trace-covered in the Render Bundle’s `trace_map`.

If a candidate factual statement is not traceable, Architect-Render MUST deterministically:
- omit it, OR
- mark it uncertain (only when upstream encodes uncertainty/ambiguity), OR
- refuse (under strict policies), with a deterministic refusal code.

---

## 6) Feedback rule: feedback does not mutate truth

User corrections/signals MUST NOT directly edit or delete canonical truth. Feedback becomes **governed work** (Orgo Case/Task or equivalent). Only a subsequent validated+compiled build may produce a new Exchange that incorporates the correction.

---

## 7) Integrity and verification (truth is also “verifiable”)

### 7.1 Fail-closed verification
If an artifact declares required hashes/signatures, verification failure MUST be treated as a hard error: the artifact MUST NOT be accepted for compilation, execution, distribution, or rendering.

### 7.2 Distribution boundary is gated
Runtime Pack distribution/activation is a strict boundary:
- verify hashes/signatures fail-closed,
- activate atomically,
- rollback deterministically to last-known-good or pinned.

---

## 8) What “source of truth” means in practice

A statement is “true in the ecosystem” only if:

1) It exists in an Exchange (or in a Runtime Pack provably derived from an Exchange), and  
2) Its identity/integrity verifies under declared policies (fail-closed), and  
3) Any downstream representation (rendered text, UI, reports) can trace it back to canonical statements/evidence via `trace_map`.

This is the combined invariant set: truth boundary + immutability + no-new-facts + fail-closed verification + governed feedback + auditability.

---

## 9) Conformance checklist (minimum)

Implementations claiming conformance MUST prove:

- “No compile on fail” is enforced.
- Exchange updates are versioned artifacts, not in-place edits.
- Render outputs do not introduce new facts and provide required trace coverage (or omit/uncertain/refuse deterministically per policy).
- Verification is fail-closed for declared required signatures/hashes.
- Feedback cannot mutate Exchange; it must open governed work.

---

## 10) Appendix: boundary diagram (conceptual)

**Inputs → Claim-IR → Resolved Claim-IR → Validation Gate → Compile → Exchange (Truth) → Runtime Pack (Derived) → Render / Execute / Distribute**

Truth starts at **Exchange**, not earlier.
