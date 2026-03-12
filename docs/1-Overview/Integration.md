# Integration

This section explains how external systems integrate with the kOA Digital Ecosystem—without requiring you to learn internal schemas or deep implementation details.

## What “integration” means here

An integration is any system that:
- produces inputs that flow toward canonical truth,
- implements a pipeline stage under Orgo control,
- consumes runtime knowledge (Runtime Packs) through Konnaxion/Malkuth,
- or needs to interoperate with Kristal v4 artifacts at the contract boundary.

## Non-negotiable integration rules

- **Do not duplicate Kristal contracts.** Kristal v4 is the sole normative source for Kristal artifact schemas/contracts. kOA points to the pinned Kristal source-of-truth.  
- **Pin Kristal v4.** Never integrate against “latest”; always integrate against a pinned Kristal version + pinned schema pointers.  
- **Fail-closed.** If verification/validation cannot be proven, do not proceed. No “best effort” activation or truth compilation paths.  
- **Opaque references.** kOA operational records store opaque references to Kristal artifacts (IDs + manifest refs), not re-parsed Kristal internals.

## Integration entry points (choose what you are)

### 1) Input producer (upstream)
You provide raw inputs/evidence that will eventually become canonical through validation + compilation. Expect to provide stable provenance and replayable snapshots (so builds can be reproduced).

### 2) Stage service (pipeline component)
You implement a stage (e.g., extractor, resolution adjunct, validation adjunct) that Orgo orchestrates. Your contract is: accept explicit artifact references, produce explicit artifacts, be retry-safe, and emit stable reason codes.

### 3) Distribution/runtime consumer
You run Konnaxion (or consume its outputs) to deliver offline-first Runtime Packs to runtime environments. Your contract is: verify-before-activate, atomic activation, deterministic rollback, and auditable events.

## Where the authoritative contracts live

- **Kristal v4 boundary contracts (pinned):** see [Integration-Kristal-v4](Integration-Kristal-v4)  
- **kOA operational expectations (gates, rollout, rollback, evidence):** see [Operations](Operations)  
- **Practical checklists/patterns:** see Integrator Guide (optional) in the main docs set (not duplicated in the wiki)

## Quick start checklist

1. Confirm the pinned Kristal v4 dependency used by this repo.
2. Produce/consume Kristal artifacts using the pinned Kristal schema pointers.
3. Apply the kOA profile defaults (verification/activation expectations).
4. Run the conformance suite (determinism + fail-closed + rollback).
5. If migrating legacy formats, use the documented legacy-compat window and deprecation path.

## Telemetry expectations (high level)

Integrations should emit:
- build/release correlation IDs (from Orgo),
- stage timing and stable error codes,
- references to logs/traces (not raw blobs in-band),
- activation/rollback outcomes and health signals (if distribution/runtime).

## Security expectations (high level)

- Never leak secrets into logs/telemetry.
- Treat verification (schema/integrity/signature/compatibility) as mandatory gates.
- Prefer least privilege and audit-grade logging for privileged actions.

## Next pages

- [Integration-Kristal-v4](Integration-Kristal-v4)
- [Integration-External-systems](Integration-External-systems)