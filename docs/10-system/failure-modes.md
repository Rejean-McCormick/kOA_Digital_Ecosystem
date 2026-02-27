# Failure Modes

**Normative for kOA:** YES
**External normative reference:** Kristal v4 (pinned)

This page defines the kOA system-level failure taxonomy, how failures propagate across nodes, and what constitutes safe behavior at the kOA truth boundary. Kristal v4 defines the normative artifact validity rules; kOA defines how the system reacts when those rules are not met.

## Principles

* **Fail closed at the truth boundary.** Any artifact that fails Kristal verification, schema validation, or determinism checks must not be compiled, promoted, activated, or served.
* **Determinism over availability.** If determinism cannot be guaranteed, kOA must halt promotion/activation rather than proceed with “best effort.”
* **Separate “content invalid” from “system impaired.”** Content failures are handled via quarantine and repair; infrastructure failures are handled via retry/backoff, degradation, and operator escalation.
* **No implicit mutation.** Runtime feedback, overrides, or emergency controls cannot silently change canon; they must be recorded as governed kOA work items.

## Failure classes

### F1 — Input acquisition failures

**Description:** Missing, unreadable, or inconsistent source inputs before Claim-IR formation.

**Examples**

* Input snapshot missing/expired
* Source connector unavailable
* Permission/credential failure
* Partial fetch leading to non-reproducible inputs

**Required behavior**

* Mark run as blocked; do not emit candidate artifacts
* Retry only if inputs can be fetched deterministically (same snapshot identity)
* Record an Orgo Task for remediation (credential fix, snapshot regeneration)

**Operator signals**

* Elevated fetch errors, timeouts, checksum mismatch on snapshots

---

### F2 — Claim formation and resolution failures

**Description:** The system cannot produce a well-formed Claim-IR or cannot resolve it to a fully grounded, policy-compliant Resolved Claim-IR.

**Examples**

* Claim-IR schema invalid (kOA-side pre-check)
* Missing required citations/anchors in resolution
* Policy gate rejects resolution (e.g., insufficient evidence, disallowed sources)

**Required behavior**

* Do not proceed to validation/compile
* Emit a failure report (kOA build record) with actionable error codes
* Create Orgo work item (repair claim, adjust mandate, fix input coverage)

**Operator signals**

* Increased policy rejections
* Repeated failures on the same mandate indicating systemic rule breakage

---

### F3 — Kristal verification failures (truth boundary)

**Description:** Any failure to verify Kristal artifacts or their integrity. This is the hard gate.

**Examples**

* Signature invalid or signer not trusted
* Hash mismatch (artifact tampering or corruption)
* Canonicalization mismatch
* Schema nonconformance for Kristal artifacts
* Traceability requirements not satisfied (missing references, broken links)

**Required behavior**

* Quarantine the artifact set (do not compile, do not publish)
* Prevent activation and rollback if already active
* Raise high-severity alert if failure occurs in a promoted/active channel
* Record exact verification results and offending artifact IDs in the build record

**Operator signals**

* Verification error spikes
* Trust store changes correlated with failures
* Artifact store corruption indicators

---

### F4 — Determinism failures

**Description:** The same declared inputs and policies do not reproduce identical outputs.

**Examples**

* Non-deterministic compilation output
* Runtime Pack differs across rebuilds with identical inputs
* Environment-dependent variation (locale/timezone, unordered maps, nondeterministic dependency versions)

**Required behavior**

* Block promotion
* Require reproducibility replay in a clean environment
* If repeated: open incident and treat as systemic defect, not content defect

**Operator signals**

* Rebuild mismatch rate
* Differences clustered by runner image/version

---

### F5 — Compilation and packaging failures

**Description:** Compiler cannot produce a valid runtime output from a verified/validated input set.

**Examples**

* Compiler crash
* Unsupported feature in the target runtime
* Missing dependency in build environment
* Packaging step fails (bundle assembly)

**Required behavior**

* Fail build; do not publish
* Retry only with identical toolchain and pinned dependencies
* Escalate if failures exceed threshold or affect multiple mandates

**Operator signals**

* Build failures correlated with toolchain upgrades
* Increased package assembly errors

---

### F6 — Distribution and activation failures (Konnaxion)

**Description:** Artifacts cannot be distributed, verified, or activated in target environments.

**Examples**

* Target cannot fetch pack
* Activation preflight fails (integrity, compatibility, policy mismatch)
* Partial rollout stalls
* Rollback fails or is unavailable

**Required behavior**

* Do not activate partially unless rollout policy explicitly allows staged activation
* Prefer automatic rollback on activation failure
* Maintain last-known-good active pack per channel/environment
* Ensure activation is idempotent and auditable

**Operator signals**

* Rollout stuck percentage
* Activation failure rate by environment
* Rollback latency increases

---

### F7 — Runtime enforcement and safety failures

**Description:** Active runtime violates declared constraints or cannot enforce gating logic.

**Examples**

* Policy enforcement module disabled or bypassed
* Missing trace map at runtime where required
* Runtime uses stale pack without declaring it
* Safety filter failure (must degrade to safe responses)

**Required behavior**

* Degrade to safe mode (read-only, deny, or minimal behavior as defined by kOA policy)
* Trigger emergency rollback if policy requires
* Generate incident ticket with high severity

**Operator signals**

* Missing policy decision logs
* Drift between active pack ID and reported pack ID
* Increased unsafe output detections

---

### F8 — Observability and audit failures

**Description:** The system cannot produce sufficient logs/records to reconstruct what happened.

**Examples**

* Missing build record or release record
* Missing activation events
* Incomplete verification logs

**Required behavior**

* Treat as a release blocker for regulated/strict channels
* Continue running last-known-good but halt new promotions
* Escalate to operators

**Operator signals**

* Logging pipeline errors
* Missing event streams, gaps in audit timeline

---

## Propagation rules (how failures move through the system)

* **F1–F2** are “pre-boundary” and should not produce promotable outputs.
* **F3–F4** are “boundary/integrity” and must block promotion/activation and trigger quarantine.
* **F5** blocks publication but does not necessarily indicate content corruption.
* **F6–F7** require environment-specific safety behavior (staged rollback, safe mode).
* **F8** blocks promotions in strict channels because auditability is part of safety.

## Severity and required response

* **SEV-1:** Any F3/F7 impacting an active channel in production, or rollback unavailable when required.
* **SEV-2:** Widespread F4 determinism failures, repeated F6 activation failures across environments.
* **SEV-3:** Elevated F1/F5 rates, localized F6, or intermittent F8.
* **SEV-4:** Single-mandate issues with clear remediation.

## Required artifacts (kOA-owned) for failure handling

* **Build Record:** captures stage outcomes, exact error codes, artifact IDs, and verification results.
* **Release Record:** captures what was promoted, where, and under which policies.
* **Orgo Task/Case:** tracks remediation actions with ownership and approval.

(Definitions live in `30-artifacts/`.)
