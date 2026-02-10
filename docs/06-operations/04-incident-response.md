# Incident Response

## Purpose
Provide a consistent operational playbook to detect, triage, contain, remediate, and learn from incidents across the ecosystem (Orgo, SenTient, Kristal compiler, Konnaxion, Architect, SwarmCraft). :contentReference[oaicite:0]{index=0}

## Core principles
- Fail fast, and **fail closed** wherever integrity is involved. :contentReference[oaicite:1]{index=1}
- Preserve ambiguity rather than blocking pipelines indefinitely (where policy allows). :contentReference[oaicite:2]{index=2}
- Quarantine poison inputs (DLQ) instead of infinite retry or silent drop. :contentReference[oaicite:3]{index=3}
- Bound resource usage (timeouts/quotas) to protect offline and multi-tenant systems. :contentReference[oaicite:4]{index=4}
- Prefer immutable releases (Exchange + Runtime Pack). Treat new data as a new build. :contentReference[oaicite:5]{index=5}

## Severity levels
- **P0**: Canon integrity or tenant isolation at risk; fail-closed bypass; widespread incorrect output.
- **P1**: Production pipeline blocked for a tenant/channel; distribution/activation failing.
- **P2**: Partial degradation (latency, elevated failures, DLQ backlog) with safe fallbacks.
- **P3**: Localized issue, non-urgent, no correctness risk.

## Incident lifecycle
1) Detect
2) Triage (scope, severity, blast radius)
3) Contain (stop the bleeding)
4) Diagnose (root cause)
5) Remediate (fix + verify)
6) Recover (resume normal ops)
7) Post-incident (report + preventive actions)

## Required context (always capture)
Minimum incident record fields:
- `tenant_id`, `environment`, `channel`
- `stage` (ingest/extract/resolve/validate/compile/publish/distribute/render/execute)
- `build_id` and (when present) `kristal_id`, `pack_id`
- `case_id`, `task_id`, request/query identifiers
- primary `error_code` and bounded error message
- timestamps + attempt count if retries occurred :contentReference[oaicite:6]{index=6}
Correlation IDs must tie events end-to-end. :contentReference[oaicite:7]{index=7}

---

# Standard containment levers (by subsystem)

## Orgo (control plane)
- Pause affected workflows / stages (per-tenant if possible).
- Stop at the failing gate; do not allow stage skipping. :contentReference[oaicite:8]{index=8}
- Surface structured errors/warnings to operators. :contentReference[oaicite:9]{index=9}

## SenTient (resolution)
- Circuit breaker per tenant/endpoints; fail fast when open. :contentReference[oaicite:10]{index=10}
- If policy allows, emit “unresolved preserved” outputs with warnings; otherwise stop early with a clear failure reason. :contentReference[oaicite:11]{index=11}

## Ingest/Extract/Resolve/Validate pipeline
- Use staged queues with bounded retries and DLQ on repeated failure. :contentReference[oaicite:12]{index=12}
- DLQ items should automatically create Orgo Cases/Tasks for triage. :contentReference[oaicite:13]{index=13}

## Konnaxion (distribution facet)
- **Fail closed** on verification failures; do not activate packs. :contentReference[oaicite:14]{index=14}
- Roll back deterministically to pinned or last-known-good pack. :contentReference[oaicite:15]{index=15}
- Remain correct offline: serve active pack; avoid network-dependent trust roots. :contentReference[oaicite:16]{index=16} :contentReference[oaicite:17]{index=17}

## Architect (rendering)
- Enforce trace coverage: every factual statement must have support or be omitted/marked uncertain or fail deterministically. :contentReference[oaicite:18]{index=18}
- Use deterministic refusal/error codes and return structured status. :contentReference[oaicite:19]{index=19}

## SwarmCraft (execution)
- Validate tasks early; fail tasks rather than “improvise.” :contentReference[oaicite:20]{index=20}
- Enforce atomicity per task and emit full telemetry with correlation IDs. :contentReference[oaicite:21]{index=21}

---

# Runbooks (common incidents)

## 1) Validation failures (pipeline correctness)
**Symptom:** Elevated workflow failures at Validate stage; no new Exchange/Pack output.

**Expected behavior (hard gate):**
- If validation fails, Orgo MUST mark the workflow failed and MUST NOT compile Exchange/Runtime Pack. :contentReference[oaicite:22]{index=22}

**Triage**
- Confirm stage ordering is respected (ingest → Claim-IR → resolve → validate → compile → publish/distribute). :contentReference[oaicite:23]{index=23}
- Pull the validation report and extract structured reason codes. :contentReference[oaicite:24]{index=24}

**Containment**
- Pause only the failing tenant/channel builds.
- Route affected inputs to DLQ if the failure is input-specific. :contentReference[oaicite:25]{index=25}

**Remediation**
- Fix schema/config/policy mismatch (Binah bundle) or input issues.
- Re-run build with same inputs/config to confirm determinism (same outcome expected).

---

## 2) SenTient outage / high latency (resolution bottleneck)
**Symptom:** Resolve stage timeouts; queues backing up; retries amplifying load.

**Containment**
- Enable circuit breaker per tenant/endpoints. :contentReference[oaicite:26]{index=26}

**Fallback options**
- If policy allows: preserve unresolved ambiguity and continue (marked `status="unresolved"` + warnings + correlation IDs). :contentReference[oaicite:27]{index=27}
- Otherwise: stop early and fail the workflow with clear reason codes.

**Remediation**
- Restore dependency health, then close breaker.
- Replay a limited set of builds to validate recovery without creating retry storms.

---

## 3) Poison inputs / repeated stage failures (DLQ required)
**Symptom:** Same inputs fail repeatedly; stage queues stuck; infinite retries.

**Containment**
- Enforce bounded retries + DLQ per stage. :contentReference[oaicite:28]{index=28}

**DLQ payload requirements**
Include build/tenant/stage/input_ref/error_code/attempt_count/timestamps. :contentReference[oaicite:29]{index=29}

**Triage loop**
- DLQ item opens Orgo Case/Task for operators; outcomes are fix input (requeue), adjust policy (if non-core), or reject with recorded reason. :contentReference[oaicite:30]{index=30}

---

## 4) Pack verification failure (integrity incident)
**Symptom:** Activation fails; signature/hash mismatch; clients stuck or cannot update.

**Invariant**
- If signatures/hashes are declared, Konnaxion MUST verify before activation and MUST fail closed on any verification failure. :contentReference[oaicite:31]{index=31}

**Containment**
- Block activation for the affected channel/tenant.
- Freeze distribution to prevent propagation.

**Recovery**
- Roll back to pinned or last-known-good pack (deterministically). :contentReference[oaicite:32]{index=32}
- Verify trust roots are pinned and available offline (no activation-time network dependency). :contentReference[oaicite:33]{index=33}

**Remediation**
- Rebuild and re-sign pack; re-verify manifest signature(s); optionally verify bundle file hashes in correct order. :contentReference[oaicite:34]{index=34}

---

## 5) Render correctness failures (trace / “new fact risk”)
**Symptom:** Users see refusals; output missing; or incorrect/untraceable assertions detected.

**Invariants**
- Every factual statement must have trace support; otherwise omit/mark uncertain/fail deterministically. :contentReference[oaicite:35]{index=35}
- Architect must implement deterministic refusal/error codes (e.g., UNVERIFIED_INPUT, MISSING_TRACE_IDS, POLICY_VIOLATION_NEW_FACT_RISK). :contentReference[oaicite:36]{index=36}

**Triage**
- Identify whether failure is:
  - input not validated (pipeline issue),
  - missing stable IDs/evidence pointers (data contract issue),
  - projection mismatch (configuration issue),
  - attempted unsupported “fact” output (template/spec issue). :contentReference[oaicite:37]{index=37}

**Containment**
- Disable the specific template/render_kind for affected tenant/channel.
- Prefer refusal over “best-effort facts.”

**Remediation**
- Fix upstream statement IDs/evidence pointers or adjust templates to render ambiguity explicitly. :contentReference[oaicite:38]{index=38}

---

## 6) Execution incidents (SwarmCraft failures)
**Symptom:** Tasks failing, partial outputs, missing artifacts, dependency/order bugs.

**Invariants**
- Task dependencies/order must be honored; tasks are immutable once issued; failures explicit; telemetry mandatory. :contentReference[oaicite:39]{index=39}
- Atomicity per task; log start/end/outcome and correlation IDs; capture output locations. :contentReference[oaicite:40]{index=40}

**Containment**
- Pause execution for the affected workflow/tenant.
- Fail tasks fast when prerequisites (files, agent availability) are missing. :contentReference[oaicite:41]{index=41}

**Remediation**
- Fix executor availability/tool adapters or task envelope issues.
- Re-run tasks deterministically where possible; ensure outputs match spec (format/location).

---

# Non-mutating feedback rule (important)
Operational signals and user feedback MUST NOT mutate Exchange directly; they must become governed work (new Cases/Tasks) triggering a new pipeline run. :contentReference[oaicite:42]{index=42} :contentReference[oaicite:43]{index=43}

---

# Post-incident requirements
1) Incident report (timeline, blast radius, root cause, fix).
2) Add/strengthen conformance tests:
   - tamper pack → fail-closed,
   - “new fact without trace” → refuse,
   - validation fail → no compile. :contentReference[oaicite:44]{index=44}
3) Update operational guidance using the standard template (concept → problem → solution → when → pitfalls). :contentReference[oaicite:45]{index=45}
4) If invariants/contracts changed, record an ADR and version the relevant schemas/contracts.
