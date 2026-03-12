# Kristal v4 integration

Kristal v4 is the **normative source** for truth-compilation contracts in this ecosystem (schemas, canonicalization rules, and integrity targets). kOA integrates with Kristal by **pinning** a specific Kristal version, enforcing **fail-closed gates**, and recording **opaque references** to Kristal artifacts (not re-specifying them).

## What this integration is (in one sentence)

kOA uses Kristal v4 to turn validated inputs into **canonical truth + runtime packs**, while kOA owns orchestration, governance, release/rollback, and operational evidence.

## Integration principles

1. **Pin Kristal (no floating “latest”)**
   - Builds and releases must reference a specific Kristal version.
2. **Do not duplicate Kristal contracts**
   - This wiki does not restate Kristal schemas or canonicalization rules.
3. **Fail-closed boundaries**
   - If validation/verification cannot be completed, the system must not compile or activate.
4. **Opaque references across the boundary**
   - kOA-native records store Kristal artifact IDs/manifests/references, not Kristal internals.

## What flows between kOA and Kristal (conceptual)

### Into Kristal
- Pinned inputs (snapshots + policy context)
- Resolved/validated material required for compilation
- Compilation intent (what pack(s) to build)

### Out of Kristal
- Canonical truth artifacts (Exchange)
- Runtime artifacts (Runtime Pack + manifests)
- Evidence artifacts (Validation Report and related outputs)

kOA treats these as **Kristal-owned artifact families**.

## Where the contracts live

- **Kristal schemas/specs:** in the pinned Kristal v4 dependency (authoritative)
- **kOA integration profile:** the kOA rules for how Kristal is used (pinning, gating, release expectations)
- **Conformance tests/checks:** the minimum checks required for a build/release to be considered valid in kOA

This page stays at the “what/why” level; see `Integration.md` for the overall integration map and `Operations.md` for release/rollback behavior.

## Pinning and upgrades (operator view)

### Pinning
- The Kristal version used by a build is recorded as part of the build/release evidence.
- Different channels/environments may intentionally run different pinned versions during staged rollouts.

### Upgrading
Upgrades are treated like a controlled change:
1. Pin the new Kristal version (explicitly).
2. Run conformance checks.
3. Build packs in a pre-production channel.
4. Canary rollout with monitoring.
5. Promote to stable if gates remain green; rollback if integrity or behavior deviates.

## Conformance expectations (plain language)

A conformant kOA↔Kristal integration should ensure:
- **Typed artifact boundaries** (no hidden shared state)
- **Schema validation** against the pinned Kristal version
- **Fail-closed behavior** at compilation and activation boundaries
- **Deterministic builds** for pinned inputs/policies/configuration
- **Auditable evidence** (records that show what ran and what was promoted)

## What kOA owns vs what Kristal owns

### kOA owns
- Orchestration and gates (control plane)
- Governance workflow and operational evidence (build/release records)
- Distribution/activation/rollback mechanics
- Observability and incident process

### Kristal owns
- Normative schema contracts for Kristal artifacts
- Canonicalization rules and integrity targets for truth artifacts and runtime packs
- The artifact family definitions themselves

## Common integration failures (non-technical)

- “Drift”: Kristal version was not pinned consistently across build/release.
- “Contract mismatch”: artifacts validated against the wrong Kristal version.
- “Best-effort activation”: pack was activated without full verification.
- “Leaky boundary”: downstream layers treat non-canonical outputs as truth.

## Related pages

- `Integration.md` (overall integration map)
- `Operations.md` (release/rollback behavior)
- `Artifacts.md` (artifact families and why they exist)