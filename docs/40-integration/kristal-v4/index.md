# Kristal v4 Integration (kOA)

**Purpose:** Define how the kOA Digital Ecosystem integrates with **Kristal v4** without duplicating Kristal’s normative contracts.  
**Rule:** Kristal artifacts (Claim-IR, Resolved Claim-IR, Validation Report, Exchange, Runtime Pack, manifests, signatures) are **normatively defined by Kristal v4**. kOA only defines:
- how it **pins** the Kristal dependency,
- which **profiles/policies** it chooses,
- which **gates** it enforces (fail-closed),
- which **conformance checks** are required,
- how it handles **legacy compatibility**.

## What lives in this section

- `pinned-dependency.md`  
  How kOA pins Kristal v4 (commit/tag), how references are made, and how updates are managed.

- `contract-pointers.md`  
  The authoritative links/paths into Kristal v4 docs/schemas for each Kristal artifact used by kOA.

- `koa-profile.md`  
  kOA’s selected profiles/policies for canonicalization, signing, validation strictness, distribution/activation behavior.

- `conformance.md`  
  Required tests and gate checks (CI + runtime) to be considered conformant.

- `legacy-compat.md`  
  Allowed legacy spellings/aliases on input (if any), and kOA’s deprecation windows.

## Integration invariant (kOA)

kOA MUST NOT:
- redefine Kristal schemas/contracts in kOA docs,
- publish or validate Kristal artifacts against kOA-owned schemas,
- accept non-verifying or “best effort” integrity checks for activation or truth compilation paths.

kOA MUST:
- treat Kristal v4 schemas as the contract boundary,
- fail-closed on verification and gate failures,
- record opaque references to Kristal artifacts in kOA operational records (Build/Release).

## Quick start

1. Pin Kristal v4 (see `pinned-dependency.md`).  
2. Implement artifact production/consumption using Kristal v4 schemas (see `contract-pointers.md`).  
3. Apply the kOA profile defaults (see `koa-profile.md`).  
4. Run conformance suite (see `conformance.md`).  
5. Configure legacy acceptance window if migrating from older artifacts (see `legacy-compat.md`).