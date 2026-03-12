# Operations — Builds

This page explains how kOA runs **builds** in operational terms (no schema deep-dives).

A **build** is one governed pipeline run that *attempts* to produce:
- a **Kristal Exchange** (canonical truth), and
- a **Runtime Pack** (portable/offline runtime artifact),
with full audit evidence.

---

## 1) What a build is (and is not)

A build is:
- reproducible under pinned inputs/config/policies,
- gated (deterministic pass/fail),
- auditable (a Build Record exists for every attempt).

A build is not:
- a “best effort” job that partially publishes on failure,
- a way to hot-edit canon (canon changes require a new build).

---

## 2) Preconditions (before you start)

A build must have:
- a pinned **Blueprint** (schemas/config/policies),
- a pinned **Mandate** when required (governance/approvals),
- a defined input surface (sources/tenants/scope),
- a place to persist artifacts and operational records.

Operational rule: Orgo must be able to persist a **Build Record** for every attempt.

---

## 3) Stage spine (normative ordering)

Orgo enforces this ordered spine:

1. **Ingest** → input snapshots recorded  
2. **Extract** → Claim-IR produced  
3. **Resolve** → Resolved Claim-IR produced  
4. **Validate** → Validation Report produced (**acceptance gate**)  
5. **Compile** → Exchange + Runtime Pack produced  
6. (Optional in build flow) **Distribute / Activate** happens via Release operations

Hard rule: **If Validate fails, the build stops and must not Compile** (“no compile on fail”).

---

## 4) What gets recorded (Build Record)

Every build attempt produces a **Build Record** that allows someone to reconstruct:

- exactly which inputs were used,
- what was pinned (blueprint/mandate/policies/tool identities),
- which stages ran and in what order,
- the gate decisions (especially validation),
- which outputs were produced (or explicitly absent on failure).

The Build Record is the primary operational evidence for:
- audits,
- reproducibility checks,
- release eligibility decisions,
- incident triage.

---

## 5) Gates (what can block the build)

### 5.1 Validation gate (hard)
- Deterministic pass/fail.
- On FAIL: stop; persist the Validation Report reference; open a triage Case/Task if required by policy.

### 5.2 Schema/contract checks (hard where applicable)
- Produced artifacts must be schema-valid against the pinned contracts (Kristal-owned for Kristal artifacts; kOA-owned for kOA artifacts).

### 5.3 Integrity checks (policy-defined)
- If hashes/signatures are declared, verification is fail-closed (no bypass).

---

## 6) Outputs (what “success” means)

A successful build produces references to:
- Validation Report (PASS),
- Exchange + Exchange Manifest,
- Runtime Pack + Runtime Pack Manifest,
and records them in the Build Record.

A failed build produces:
- Build Record (FAIL),
- the most recent stage outputs (as refs),
- deterministic failure reason codes,
- **no** Exchange/Runtime Pack outputs if validation did not pass.

---

## 7) Reruns and idempotency (operational expectations)

- Re-running a build with the **same pinned context** should yield identical (or canonically identical) outputs.
- Retries should not create ambiguous “double outputs”: the Build Record must make it clear which attempt/output is authoritative.
- If determinism cannot be met, move nondeterminism upstream and freeze it as an input artifact.

---

## 8) Common failure classes (operator view)

- Ingest: missing provenance, access policy rejection, snapshotting failure
- Extract/Resolve: schema-invalid outputs, unstable ordering, unresolved ambiguity mishandled
- Validate: deterministic rejection (expected), or unstable rejection (investigate determinism)
- Compile: compiler failure or non-conformant output (block downstream)
- Environment/tooling: unpinned dependency drift, incompatible pins, artifact store failures

---

## 9) Operator checklist (quick)

1. Identify `build_id`
2. Retrieve the Build Record
3. Confirm pinned context: blueprint/mandate/policy/tool identities
4. Inspect stage timeline + first failing stage
5. For validation failures: retrieve Validation Report ref and failure codes
6. Confirm “no compile on fail” was respected
7. If recurring failures: open/attach Orgo Case and track remediation tasks

---

## 10) Links

- Pipeline runbook (stage-by-stage operational checks): `Operations.md` / pipeline references
- Releases (turn builds into controlled rollouts): `Operations-Releases.md`
- Rollbacks (when distribution/activation goes wrong): `Operations-Rollbacks.md`
- Artifact evidence: `Artifacts.md` → Build Record / Release Record