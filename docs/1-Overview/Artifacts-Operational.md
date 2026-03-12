# Operational Artifacts (kOA-native)

Operational artifacts are **typed payloads** used to **operate, govern, distribute, and observe** the ecosystem. :contentReference[oaicite:0]{index=0}  
They do **not** redefine Kristal artifacts; they reference Kristal outputs by opaque IDs defined in the pinned Kristal spec. :contentReference[oaicite:1]{index=1}

---

## What makes an artifact “operational”

An operational artifact exists to answer questions like:

- What work was authorized and executed?
- What inputs/policies were pinned?
- What passed/failed (and why)?
- What was rolled out, where, and what is active now?
- What was rolled back, and what evidence supports that decision?

Each kOA artifact has:
- a documented contract (human-readable),
- a JSON Schema for validation, and
- explicit versioning/compat expectations. :contentReference[oaicite:2]{index=2}

---

## Operational artifact catalog

### 1) Governance artifacts (Orgo)

#### Orgo Case
A **case** is the top-level unit of governed operational work (incident, remediation, investigation, rollout review, etc.). It provides:
- a stable identifier and status,
- ownership and routing,
- links to evidence and related tasks.

#### Orgo Task
A **task** is an executable unit of work under constraints (tool/agent/human/hybrid), with pinned inputs and declared expected outputs.
Tasks are the primary way the system turns “needed action” into auditable execution and evidence linkage. :contentReference[oaicite:3]{index=3}

**Typical usage**
- “Investigate repeated validation failures”
- “Re-run build under pinned context”
- “Rollback stable channel to last-known-good”
- “Rotate trust material and verify activation”

---

### 2) Pipeline operational records

#### Build Record
A **Build Record** is the audit-grade record of a pipeline run. It must allow an auditor to reconstruct:
- exactly which input snapshots were used,
- what pinned context was applied (blueprint/mandate/policy/toolchain identity),
- which stages ran and their outcomes,
- gate decisions (especially validation),
- which outputs were produced (Kristal Exchange + Runtime Pack refs). :contentReference[oaicite:4]{index=4}

**Key invariant**
- **No compile on fail**: if validation fails, the Build Record must not claim Exchange/Runtime Pack outputs for that attempt. :contentReference[oaicite:5]{index=5}

**Why it exists**
- Reproducibility (“same pinned context → comparable results”)
- Gate accountability (“what failed, where, and why”)
- Release eligibility input (releases start from a PASS build)

---

#### Release Record
A **Release Record** captures **distribution intent** and **rollout state** for publishing verified build outputs (e.g., Runtime Packs) to channels/cohorts. :contentReference[oaicite:6]{index=6}  
It binds:
- what is being released (references to build outputs),
- where it is released (channels/cohorts),
- how it is released (policy + rollout strategy),
- what happened (verification/activation outcomes, timestamps, operators, reasons). :contentReference[oaicite:7]{index=7}

**Important boundary**
- A Release Record does not mutate canonical truth; it is operational governance used by Orgo/Konnaxion. :contentReference[oaicite:8]{index=8}

---

### 3) Policy / mandate

#### Mandate Bundle
A **Mandate Bundle** is a pinned, versioned bundle of governance inputs that define what the ecosystem is allowed to do and how it must behave. :contentReference[oaicite:9]{index=9}  
It exists to:
- provide a single content-addressed governance package referenced by builds/releases/audits,
- enable reproducible policy decisions,
- separate governance configuration from runtime execution and from canonical truth artifacts. :contentReference[oaicite:10]{index=10}

---

### 4) Distribution / runtime

#### Konnaxion State
**Konnaxion State** captures the local distribution/activation state of a Konnaxion instance (or cluster) in a structured, auditable form. :contentReference[oaicite:11]{index=11}  
It is designed to:
- provide a consistent view of what is installed/active/pinned,
- support deterministic rollback decisions,
- correlate activation outcomes with health signals. :contentReference[oaicite:12]{index=12}

**When it’s emitted**
Konnaxion updates this record on verify/activate attempts (success or fail), rollback start/completion, pin/unpin changes, and optionally periodic heartbeat. :contentReference[oaicite:13]{index=13}

---

## How these artifacts fit together (operator mental model)

- **Orgo Case/Task**: the governed work and evidence linkage.
- **Build Record**: what ran, under what pinned context, and what passed/failed.
- **Release Record**: how a successful build was rolled out (channels/cohorts), and what happened.
- **Konnaxion State**: what is actually installed/active/pinned in runtime environments.
- **Mandate Bundle**: the pinned policy context that constrains all of the above.

---

## Related pages

- Artifacts.md
- Operations.md
- Operations-Builds.md
- Operations-Releases.md
- Operations-Rollbacks.md