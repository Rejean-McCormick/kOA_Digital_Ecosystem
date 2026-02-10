# Determinism

## Purpose
Determinism is the backbone that makes the ecosystem auditable, reproducible, and safe to automate at scale.

In this ecosystem, “deterministic” means:
- the same validated inputs + the same declared policies + the same compiler/render versions
- produce the same artifacts and outputs (bit-identical where applicable),
- and any deviation is either explicitly declared (via profiles) or rejected.

Determinism is not “rigidity”: it’s **controlled change**.

---

## Definitions

### Deterministic behavior
Given identical inputs and identical declared parameters, the system produces identical outputs.

### Reproducible build
A third party can rebuild the same artifact using only:
- the recorded manifests,
- the referenced input snapshots,
- the declared policy selections,
and obtain the same content IDs and payload hashes.

### Fail-closed
If an artifact declares integrity material (hash/signature) and verification fails, the system stops. No “best-effort” acceptance.

---

## Why determinism is non-negotiable
Determinism enables:
- **Trust**: content IDs and signatures actually mean something.
- **Auditability**: you can prove what happened and why.
- **Safe automation**: pipelines can run without human babysitting.
- **Offline correctness**: outputs remain correct without network access.
- **Comparability across toolchains**: independent implementations can converge.

---

## The ecosystem determinism contract

### 1) Source of truth is typed artifacts, not prose
All knowledge and decisions must ultimately be expressed as schema-valid, typed artifacts with explicit versions.

Core artifact chain (conceptual):
1. Claim-IR (proposals + uncertainty + evidence pointers)
2. Resolved Claim-IR (resolution outputs + ambiguity preserved)
3. Validation Report (deterministic pass/fail + stable codes)
4. Exchange (canonical, content-addressed knowledge artifact)
5. Runtime Pack (derived, offline-executable index)
6. Rendered Outputs (deterministic text + trace metadata)

### 2) Deterministic gates define what may proceed
This is a “no-compromise” flow:
- If validation fails, compilation stops.
- If integrity verification fails, activation stops.
- If a renderer cannot trace a statement, it must omit, mark uncertainty (when present upstream), or fail.

### 3) Determinism is declared, recorded, and testable
Any behavior that affects outputs must be:
- chosen from declared/allowed policy sets (where applicable),
- recorded in manifests,
- and covered by acceptance tests / golden vectors.

If a build cannot guarantee determinism, it must not claim core conformance.

---

## Determinism by subsystem

### Orgo (workflow control plane)
Orgo enforces deterministic gating at the workflow level:
- orchestrates stage transitions,
- blocks “compile/publish” when validation fails,
- persists build correlation IDs and audit trails,
- ensures the pipeline is contract-driven (schemas and invariants over ad-hoc behavior).

**Principle:** Orgo never “pushes through” a failed gate.

### SenTient (resolution)
SenTient must be deterministic in *structure* and ordering:
- stable field naming and typing,
- deterministic sorting rules and tie-breakers,
- deterministic top-K truncation when used (K declared),
- deterministic literal normalization output shapes (with explicit unit/precision),
- explicit ambiguity and error signaling (never silently drop claims).

**Principle:** ambiguity is preserved, not papered over.

### Kristal compiler (Exchange + Runtime Pack)
Deterministic compilation requires:
- explicit input snapshots (what was compiled),
- explicit compiler identity (name/version) and config hash,
- explicit canonicalization profile,
- explicit policy selections affecting output semantics and indexing,
- deterministic IDs and payload hashes.

Runtime Packs claiming core conformance must explicitly declare determinism.

**Principle:** “same inputs + same policies ⇒ same IDs”.

### Architect (rendering)
Architect rendering is a pure function over validated inputs:
- deterministic outputs given identical validated bundles and template/profile versions,
- no network dependency required to be correct,
- no probabilistic sources used as factual inputs.

Hard constraint: **no new facts**.
Architect must not introduce any factual assertion that isn’t supported by at least one input statement.

Architect must produce:
- rendered text
- plus a trace map linking every factual statement back to statement IDs and evidence pointers.

**Principle:** if it can’t be traced, it can’t be stated.

### Konnaxion (distribution + offline execution)
Konnaxion enforces determinism at the “activation boundary”:
- verify hashes/signatures before activating artifacts when integrity is declared,
- fail closed on verification mismatch,
- apply versioning / rollback / downgrade rules deterministically per policy.

**Principle:** a pack is either verified + policy-compliant, or not activated.

---

## What is allowed to vary (and how)

### Controlled nondeterminism must be explicit
Some fields may legitimately vary (e.g., timestamps, operational metrics), but they must be:
- excluded from identity/hashing surfaces by explicit rule or profile,
- and never allowed to alter factual content.

### Style variation is not factual variation
If stylistic variation exists (e.g., micro-planning), it must be:
- explicitly enabled and versioned,
- constrained so it cannot alter factual assertions,
- and ideally excluded from any “factual equivalence” hashing surface.

---

## Implementation checklist

A component is “determinism-compliant” if it can answer:
1. What are the *exact* typed inputs it consumes?
2. What are the *exact* typed outputs it emits?
3. Which policies/parameters affect outputs?
4. Where are those policies recorded (manifest fields)?
5. What is the acceptance gate, and what happens on failure?
6. How do we prove equivalence across rebuilds (hashes, vectors, reports)?
7. Where do we fail closed, and where do we ignore unknown fields safely?

---

## Summary
Determinism is the ecosystem’s alignment mechanism:
- typed artifacts define reality,
- deterministic gates define progression,
- canonical identity defines comparability,
- rendering is traceable and fact-closed,
- distribution is verification-closed.

This is what makes the ecosystem “tight”.
