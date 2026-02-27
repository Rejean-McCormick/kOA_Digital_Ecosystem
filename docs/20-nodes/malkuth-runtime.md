# 10 — Malkuth: Runtime (Offline Serving + Execution Substrate)

## Purpose
**Malkuth Runtime** is the ecosystem’s **local, offline-capable serving and execution substrate**. It loads the **active Runtime Pack** (as activated by Konnaxion), exposes deterministic query/lookup interfaces, and executes pack-defined routines in a controlled environment.

Malkuth is **downstream of the truth boundary**. It must not create or mutate canonical truth. It only serves and operationalizes **derived artifacts**.

---

## Responsibilities

Malkuth MUST:
- Load and serve the **active Runtime Pack** selected by Konnaxion.
- Provide deterministic query/lookup over pack payloads (indexes, embeddings, tables, rules, templates).
- Execute pack-defined routines (if present) with pinned dependencies and stable behavior.
- Enforce isolation and resource controls (sandboxing, quotas, timeouts).
- Emit structured telemetry about queries/executions (without mutating canon).
- Support offline operation and stable behavior under network absence.

Malkuth MUST NOT:
- Fetch non-pinned external data during deterministic operations (unless explicitly configured as a non-deterministic mode and recorded upstream).
- Modify Exchange or pack payloads in place.
- Activate or roll back packs (Konnaxion owns activation).

---

## Inputs

### Primary input
- **Active Runtime Pack** (payload + manifest), provided via Konnaxion’s activation mechanism.

### Secondary inputs (optional)
- Local runtime configuration (feature flags, resource budgets, allowed interfaces).
- Local policy overlays (only if explicitly permitted and recorded; must not affect canonical IDs).

---

## Outputs

- Query results (deterministic, traceable to pack version and query inputs).
- Execution results (task outputs, logs, metrics; optionally artifacts for Orgo ingestion).
- Telemetry events to Orgo (or local store) for observability and governance.

---

## Interfaces

### 1) Pack mount interface (from Konnaxion)
Konnaxion provides Malkuth with:
- Active pack identity (pack_id / version)
- Verified payload location(s)
- Activation epoch or activation record reference
- Optional compatibility metadata (runtime ABI level)

Malkuth MUST refuse to start if the active pack is missing or fails runtime compatibility.

### 2) Query interface (to consumers)
Consumers (Architect-Render, local apps, SwarmCraft, tooling) call:
- `GET /runtime/info` → active pack identity + runtime version
- `POST /runtime/query` → deterministic queries over pack indexes/tables
- `POST /runtime/lookup` → point lookups by key/entity id
- `POST /runtime/search` → deterministic search over pack-provided indexes (if present)

All responses MUST include:
- active pack identity
- query parameters hash (or echoed request id)
- deterministic error codes on failure

### 3) Execution interface (optional; to SwarmCraft / Orgo)
If Malkuth runs pack-defined routines:
- `POST /runtime/execute` with:
  - routine id
  - inputs (content-addressed if applicable)
  - execution policy (timeout, memory, CPU budget)
  - required determinism mode (strict / best-effort)

Execution outputs MUST include:
- routine id + version
- active pack identity
- exit code + deterministic error code mapping
- logs/metrics (structured)

---

## Invariants (Non-negotiable)

1) **No new facts**
Malkuth must not introduce new canonical truth. Any outputs are **derived** and must be attributable to:
- active pack payloads, and
- explicit query/execute inputs.

2) **Fail-closed on integrity and compatibility**
If the active pack is not verified/compatible, Malkuth must refuse to serve.

3) **Deterministic serving**
For a given (pack_id, query input, runtime config in deterministic mode), outputs must be:
- bit-identical or canonically identical,
- with stable ordering and stable tie-breakers.

4) **Immutable pack payload**
Malkuth must treat pack payloads as read-only.

---

## Determinism Rules

### 1) Sorting and tie-breakers
Any results that depend on ordering MUST define:
- stable primary sort key(s)
- stable tie-breakers (e.g., lexical id ordering)

### 2) Randomness
Randomness is forbidden in strict deterministic mode.
If a routine requires randomness, it MUST accept an explicit seed and record it in the execution output.

### 3) Time and locale
Wall-clock time, locale, and environment differences MUST NOT affect deterministic outputs.
If timestamps are emitted for observability, they must not influence computed results.

### 4) Floating point / platform stability
If pack routines use float operations, Malkuth must enforce stable:
- numeric representation,
- rounding rules,
- and pinned libraries.

---

## Failure Modes

### Activation / pack load failures
- Pack missing or unreadable
- ABI/runtime version incompatibility
- Payload corruption detected at load time
- Required payload component missing (index/table not present)

### Query failures
- Invalid query schema
- Unsupported query type for this pack
- Determinism violation detected (e.g., non-stable comparator)
- Resource exhaustion (timeout / memory cap)

### Execution failures (optional)
- Routine not found
- Sandbox violation (filesystem/network access denied)
- Deterministic mode requested but routine not deterministic-capable
- Dependency mismatch (required runtime module absent)

All failures MUST produce:
- stable error codes
- active pack identity
- request id
- minimal safe diagnostics (no sensitive data leaks)

---

## Observability

Malkuth MUST emit:
- active pack state (pack_id, activation epoch)
- request counters (query/lookup/execute)
- latency histograms
- cache hit rates (if caching is enabled)
- error codes + rates
- resource usage (CPU, memory, disk IO)

Malkuth SHOULD emit:
- per-request trace spans (request id propagated end-to-end)
- deterministic mode flags (strict vs best-effort)

Telemetry MUST NOT contain:
- raw sensitive inputs unless explicitly allowed by policy
- secrets or private keys
- user-identifying data unless required and governed

---

## Security and Privacy

- Run all routines in a sandbox with least privilege.
- Deny network by default in deterministic mode.
- Restrict filesystem access to the activated pack mount + designated scratch.
- Enforce quotas and timeouts per request.
- Treat pack payload as untrusted until verified by Konnaxion (Malkuth still must validate expected structure before use).

---

## Compatibility

Malkuth MUST enforce a runtime compatibility policy:
- minimum runtime ABI version required by pack
- feature flags supported by runtime
- downgrade prevention (if required by policy)

If compatibility fails, Malkuth MUST refuse to serve (fail-closed) and surface a compatibility error code.

---

## Implementation Notes (Non-normative)

Common implementation choices:
- embed a local query engine (e.g., parquet reader + ANN index + rules engine)
- isolate routine execution via WASM/containers
- use memory-mapped indexes for performance
- maintain a small, deterministic caching layer keyed by (pack_id, request_hash)

The primary goal is not maximum performance; it is **repeatability, safety, and auditability** under offline constraints.