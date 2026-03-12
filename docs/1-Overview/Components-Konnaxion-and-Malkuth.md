# Components — Konnaxion & Malkuth

Konnaxion and Malkuth are the **distribution + runtime** pair that make “offline-first, verified knowledge” real:

- **Konnaxion (Distribution)** decides what can be installed/activated, verifies it **fail-closed**, activates it **atomically**, and rolls back **deterministically**.
- **Malkuth (Runtime)** serves deterministic queries (and optionally executes pack-defined routines) **only** over the currently active, verified pack.

---

## Konnaxion (Distribution)

### Purpose
Safely deliver Runtime Packs to environments (edge, device, offline node, cluster) and ensure that only **verified, compatible** packs become active.

### Owns
- Fetching/mirroring Runtime Packs and keeping multiple versions installed per channel/environment.
- Verification-before-activation (integrity/signature/required content).
- Compatibility checks before activation.
- Atomic activation and deterministic rollback.
- Downgrade and substitution protection (per policy).
- Offline-capable trust verification (no correctness dependency on online trust-root fetches).

### Does not own
- Defining Runtime Pack format/schemas (external).
- Deciding canonical truth (upstream of distribution).
- Rendering user-facing outputs (Architect).
- Executing tasks (SwarmCraft).

### How it works (high level)
1. **Receive** a candidate pack reference (from a release channel/index or an explicit operator action).
2. **Verify** the pack (fail-closed): parse/validate the manifest, validate integrity, validate signatures (when declared), ensure required files are present.
3. **Check compatibility** with the local runtime capabilities/profile.
4. **Activate atomically**: switch the “active” pointer from old → new with no partial states.
5. **Record state and diagnostics** for operators and Orgo (including stable reason codes on failures).
6. If needed, **rollback deterministically** (pinned known-good or last-known-good), with explicit authorization and auditability.

### Key invariants (non-negotiable)
- **Verification-before-activation** (fail-closed).
- **Compatibility is mandatory** (incompatible means no activation).
- **Atomic activation** (partial activation forbidden).
- **Deterministic rollback** (same triggers + same verified inputs → same rollback outcome).
- **Offline correctness**: trust verification must be possible offline; do not require live network trust roots for correctness.

### Operational artifact: Konnaxion State
Konnaxion emits a structured **Konnaxion State** record so operators and Orgo can see:
- what is installed,
- what is active,
- what is pinned/revoked,
- what the last attempt did and why,
- what rollback target is available.

This is an operational record; it references external artifacts by ID, it does not redefine them.

---

## Malkuth (Runtime)

### Purpose
Provide a stable runtime surface over the currently active pack:
- deterministic **query/lookup/search** over pack-provided indexes/tables, and
- optionally deterministic **execute** for pack-defined routines (policy-gated).

### Owns
- Deterministic serving over active pack payloads.
- Fail-closed refusal to serve if the active pack is not verified/compatible.
- Stable error codes and stable ordering/tie-breakers.
- Runtime safety controls (sandboxing, quotas/timeouts, deterministic-mode constraints).

### Does not own
- Activating packs (Konnaxion).
- Deciding truth or compiling canon (upstream).
- Mutating pack payloads (read-only).
- Introducing new canonical facts.

### How it works (high level)
1. Reads the **active pack pointer** as set by Konnaxion.
2. Validates that the active pack is **verified and compatible** (or refuses).
3. Serves deterministic queries over pack data structures.
4. Emits observability signals (pack identity, request ids, stable error codes, resource usage).

### Key invariants (non-negotiable)
- **No new facts**: outputs are derived only from the active pack payloads plus explicit request inputs.
- **Fail-closed** if pack verification/compatibility is not satisfied.
- **Deterministic serving**: stable outputs, stable ordering, stable tie-breakers under the same inputs/config.
- **Immutable payload**: pack is read-only.

---

## How they work together

```mermaid
flowchart LR
  R[Release/Channel Index] --> K[Konnaxion]
  K -->|verified + activated pack| A[(Active Pack Pointer)]
  A --> M[Malkuth]
  M --> C[Consumers (apps, Architect, SwarmCraft, tooling)]
  K --> S[Konnaxion State (ops/audit)]
  S --> O[Orgo / Ops / Monitoring]
````

* Konnaxion is the **gatekeeper**: it prevents corrupted, incompatible, revoked, substituted, or downgraded packs from becoming active.
* Malkuth is the **runtime contract**: it only serves from what Konnaxion has made active, and it stays deterministic and fail-closed.

---

## Operator cues (what to look at first)

* If a rollout “stalls”: check Konnaxion State for the last attempt’s **reason code** and whether verification or compatibility blocked activation.
* If runtime answers look inconsistent: confirm Malkuth is in deterministic mode and that the active pack identity matches expectations.
* If an incident occurs: prefer **explicit rollback** to last-known-good/pinned known-good, and preserve the audit trail.

