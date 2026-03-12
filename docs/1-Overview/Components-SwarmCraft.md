# SwarmCraft

SwarmCraft is the system’s **execution layer**. It runs **governed tasks** using already-validated knowledge, produces **telemetry**, and reports outcomes back to operations—without changing canonical truth by side effects.

---

## What SwarmCraft is responsible for

- Executing **approved tasks** derived from the pipeline (e.g., refresh jobs, checks, downstream actions)
- Enforcing **governance constraints** at runtime (what can run, where, and under which mandate)
- Producing **execution evidence**:
  - task outcomes (success/failure)
  - timing and performance metrics
  - structured logs and traces
  - retry/rollback signals (when applicable)
- Keeping execution **separate from truth**:
  - SwarmCraft can *use* canonical truth and packs
  - SwarmCraft does **not** create or mutate canonical truth directly

---

## What it is not responsible for

- Defining canonical truth (that’s handled upstream by validation/compilation)
- Packaging/distributing runtime packs (that’s distribution/runtime infrastructure)
- Rendering user-facing explanations (that’s rendering)
- Making release decisions (that’s operations/control plane)

---

## Inputs and outputs (high level)

### Inputs
- A **task definition** (what to do, with what limits)
- A **mandate/context** (who authorized it, policy constraints)
- References to **validated artifacts** or an **activated runtime pack**
- Execution parameters (target, schedule, scope, concurrency limits)

### Outputs
- **Task result** (success/failure + reason category)
- **Execution telemetry** (metrics/logs/traces)
- **Operational events** (alerts, escalation hooks, retry signals)
- Optional **derived artifacts** that are explicitly marked as non-canonical outputs (e.g., reports)

---

## How it works (conceptual)

1. **Receive a governed task**
   - The task arrives with an explicit authorization context and constraints.

2. **Acquire the correct truth snapshot**
   - SwarmCraft executes against a specified, activated pack or pinned artifact references.
   - It should not “wing it” with whatever is latest.

3. **Execute with guardrails**
   - Applies resource limits, timeouts, concurrency rules, and allow/deny constraints.

4. **Report outcomes**
   - Emits structured results and telemetry for operations.
   - Failures are classified so they can be routed (retry, hold, rollback, escalate).

5. **Feed back into governance**
   - Execution outcomes become inputs for future governed work (e.g., follow-up tasks, incident handling).

---

## Task classes (examples)

- **Verification tasks**
  - sanity checks, pack validation checks, drift detectors

- **Maintenance tasks**
  - cache warmups, index refresh, routine housekeeping

- **Operational workflows**
  - canary probes, rollout health checks, automated rollback triggers

- **External actions (controlled)**
  - calling downstream systems via approved connectors, with explicit scopes and rate limits

---

## Safety and governance guarantees

- **No “new truth”**
  - SwarmCraft cannot introduce facts into canonical truth. Any new knowledge must go back through the governed pipeline.

- **Pinned execution context**
  - Tasks execute against a specific pack or pinned artifact set to preserve reproducibility.

- **Fail closed by default**
  - If required context is missing or verification fails, the task should not proceed.

- **Evidence-first**
  - Every meaningful action produces telemetry and an auditable outcome record.

---

## Common failure modes (non-technical)

- **Missing or invalid execution context**
  - No approved mandate, expired authorization, or incompatible pack reference.

- **External dependency outage**
  - Downstream system is unavailable; SwarmCraft records failure category and follows retry/backoff policy.

- **Resource exhaustion**
  - Limits exceeded (time, memory, concurrency); task is terminated and reported.

- **Non-deterministic inputs**
  - Task attempted to use “latest” rather than the pinned context; should be blocked or flagged.

- **Policy violation**
  - Task attempted an action outside its allowed scope; execution is denied and logged.

---

## Operational guidance

- Prefer **small, composable tasks** with clear success criteria.
- Use **canary execution** for risky task types before broad rollout.
- Treat repeated failures as **signals for rollback/hold**, not as reasons to loosen gates.
- Keep telemetry **consistent and structured** so incidents can be triaged quickly.

---

## Quick FAQ

**Does SwarmCraft change the knowledge base?**  
No. It uses validated knowledge and reports outcomes. Any change to canonical truth must go through the governed pipeline.

**Can SwarmCraft run while offline?**  
It can execute tasks that only require an activated pack and local dependencies. Tasks that need external systems must be explicitly approved and handled as such.

**How is SwarmCraft related to releases?**  
SwarmCraft can run health checks and rollout-related tasks, but it does not decide promotions. Operations/control plane decides.