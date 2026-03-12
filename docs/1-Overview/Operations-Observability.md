# Observability

This page defines the **minimum observability** needed to operate kOA safely: detect failures early, explain outcomes deterministically, and preserve evidence for audit and incident response.

kOA treats observability as part of the pipeline: Orgo includes an explicit **Observe** stage (metrics/events/logs) before feedback/cases are created. :contentReference[oaicite:0]{index=0}

---

## Goals

1. **Fail fast, fail closed**: if the system cannot verify/validate, it must not proceed and must emit enough telemetry to diagnose without guesswork. :contentReference[oaicite:1]{index=1}  
2. **Deterministic diagnosis**: failures should produce **stable reason codes** and repeatable outcomes. :contentReference[oaicite:2]{index=2}  
3. **End-to-end correlation**: every signal ties back to a build/release/task and the active pack lineage. :contentReference[oaicite:3]{index=3} :contentReference[oaicite:4]{index=4}  
4. **Audit-grade evidence**: key actions and decisions are reconstructable (especially activation, rollback, and trust events). :contentReference[oaicite:5]{index=5}

---

## Correlation model (what every signal must reference)

At minimum, emit correlation IDs for:
- **Build** (pipeline execution) :contentReference[oaicite:6]{index=6}  
- **Release** (promotion/publish action)
- **Task** (Orgo Task identity + attempt identity) :contentReference[oaicite:7]{index=7}  
- **Runtime Pack** (pack ID / manifest ref) :contentReference[oaicite:8]{index=8}  
- **Target scope** (tenant / env / channel / cohort / device group), as applicable

Do not embed large payloads in-band; emit **references to logs/traces** and content-addressed artifacts instead. :contentReference[oaicite:9]{index=9} :contentReference[oaicite:10]{index=10}

---

## Required telemetry (baseline)

Integrations and operators should emit, at minimum: :contentReference[oaicite:11]{index=11}
- build/release correlation IDs (from Orgo)
- stage timing (start/end/fail)
- stable error codes + human-readable summaries
- references to logs/traces (not raw blobs in-band)

If you operate Konnaxion, also emit **Konnaxion State** records and activation/rollback outcomes with health signals. :contentReference[oaicite:12]{index=12} :contentReference[oaicite:13]{index=13}

---

## Signals by category

### Metrics
Use metrics to answer: “Is it healthy?” and “Is it getting worse?”

Typical examples (not exhaustive):
- Pipeline: stage success/failure rate; latency per stage; determinism/rebuild pass rate
- Distribution/runtime (Konnaxion): fetch latency/failure, verification pass/fail by reason, activation success/time, rollback frequency/triggers, cache utilization and corruption detection, active-pack drift :contentReference[oaicite:14]{index=14}
- Ingest (Chokmah): ingest req/success/failure, bytes/throughput, latency per connector/source type, retries/idempotency hits, quarantine/rejection rates by reason :contentReference[oaicite:15]{index=15}

### Logs (structured)
Logs must be structured and include correlation IDs and stable reason codes. :contentReference[oaicite:16]{index=16}  
Example ingest log fields: request id, source descriptor hash, snapshot refs, policy decisions, failure codes, diagnostics pointers. :contentReference[oaicite:17]{index=17}

### Traces
Traces should connect:
- Orgo stage execution → produced artifact refs → downstream distribution/activation attempts
- Task dispatch (Orgo Task) → executor run → result + telemetry refs :contentReference[oaicite:18]{index=18}

### Events (audit / operational)
Emit explicit events for:
- stage transitions (start/finish/fail) with reason codes :contentReference[oaicite:19]{index=19}
- key activation actions (verify/activate/rollback/pin/unpin/revoke)
- trust material events (key creation/rotation/revocation; verification failures) :contentReference[oaicite:20]{index=20}

### Operational artifacts as observability
Some observability is captured as **typed artifacts**, not just logs:
- **Konnaxion State** (installed/active/pinned, last attempt status, health, telemetry refs) :contentReference[oaicite:21]{index=21} :contentReference[oaicite:22]{index=22}  
- Orgo Case/Task audit trails for decisions and remediation work :contentReference[oaicite:23]{index=23}

---

## Release and rollback observability (must-have)

### During activation and rollback
Emit, per target: activation attempt events, preflight outcomes (pass/fail + reason codes), post-activation health results, and final active pack ID. :contentReference[oaicite:24]{index=24}

### Dashboards and alerts should include
- rollback success rate by channel/environment
- time-to-restore by incident
- rollback loops (flapping)
- divergence: reported active pack ID vs expected :contentReference[oaicite:25]{index=25}

---

## Diagnostic standards

- **Stable reason codes**: required for fail-closed behavior and repeatable triage. :contentReference[oaicite:26]{index=26}  
- **Deterministic diagnostics** at activation boundaries (verification failures must produce stable reason codes). :contentReference[oaicite:27]{index=27}  
- **Diagnostics pointers**: prefer `diagnostics_ref` / `telemetry_refs` over dumping blobs into control-plane records. :contentReference[oaicite:28]{index=28} :contentReference[oaicite:29]{index=29}

---

## Minimal checklist

- [ ] Every stage emits start/end/fail with duration and stable reason code. :contentReference[oaicite:30]{index=30}  
- [ ] Every signal includes build/release/task correlation IDs. :contentReference[oaicite:31]{index=31}  
- [ ] Konnaxion emits verification/activation/rollback signals and Konnaxion State updates. :contentReference[oaicite:32]{index=32}  
- [ ] Rollback dashboards include success rate, TTR, flapping, and drift detection. :contentReference[oaicite:33]{index=33}  
- [ ] Trust events (rotate/revoke/verify failures) are recorded and routed to security monitoring and incident response. :contentReference[oaicite:34]{index=34}