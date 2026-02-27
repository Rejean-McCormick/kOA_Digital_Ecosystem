# Glossary

**Normative for kOA:** YES (terminology usage within kOA docs)  
**External normative references:** Kristal v4 (pinned) for Kristal-defined terms; kOA does not restate Kristal contracts

**Purpose:** Define the shared vocabulary used across the kOA Digital Ecosystem docs.  
**Rule:** If a term is *normatively defined by Kristal v4*, this glossary **does not restate** the contract— it points to the Kristal source (via the pinned Kristal dependency).

## Conventions

- **Normative (Kristal):** The term’s format/semantics are defined in Kristal v4 docs/schemas.
- **Normative (kOA):** The term is defined by kOA contracts/policies/operations.
- **Informative:** Helpful interpretation only.

---

## Terms (A–Z)

### Activation (kOA)
Atomic switch from one verified Runtime Pack to another on a device/node, with deterministic rollback guarantees.

### Artifact (kOA)
A typed payload crossing a boundary between components (e.g., Case, Task, Build Record, Release Record).  
If the artifact is a Kristal artifact (Claim-IR, Exchange, Runtime Pack, Validation Report), see Kristal v4 (pinned).

### Authority Registry (Kristal)
Normative registry of trust roots and acceptance rules for signatures.  
See Kristal (pinned): `vendor/kristal/02-schemas/authority-registry.schema.json`.

### Blueprint (kOA)
The pinned bundle of schemas/config/policies that governs a build and makes downstream outputs reproducible.

### Build (kOA)
One governed pipeline run producing outputs (at minimum: Exchange + Runtime Pack), with recorded inputs/config/policies and audit linkage.

### Build Record (kOA)
Operational evidence of a build: inputs, pinned blueprint, policy selections, component identities, outputs, and gate results.

### Canonicalization (Kristal)
Deterministic normalization of JSON/signing targets used for hashing/signatures and stable identity.  
See Kristal (pinned): `vendor/kristal/01-core-spec/ids-canonicalization-hashing.md`.

### Canonical Truth (kOA)
Truth that exists only after deterministic validation and compilation into a Kristal Exchange artifact (immutable once published).

### Case (Orgo Case) (kOA)
Long-lived container for operational work, context, and governance state; parent of Tasks.

### Claim-IR (Kristal)
Proposal-format for extracted claims (schema-constrained, uncertainty + evidence explicit).  
See Kristal (pinned): `vendor/kristal/02-schemas/claim-ir.schema.json`.

### Compile (kOA)
The stage that invokes Kristal compilation to produce Kristal artifacts (Exchange + Runtime Pack) from validated inputs. Must be blocked if validation fails.

### Compatibility Policy (kOA)
Rules for accepting/activating artifacts across versions (schemas, policies, runtime constraints); used by Orgo and Konnaxion.

### Conformance (kOA)
The property of an implementation satisfying the ecosystem’s invariants and required contracts; typically enforced via tests and gates.

### Determinism (kOA)
Same pinned inputs + config + policies → identical (or canonical-identical) outputs; enforced at gates and required for reproducibility.

### Deterministic Gate (kOA)
Acceptance checkpoint that is deterministic and fail-closed (e.g., validation gate; signature verification gate).

### Distribution (kOA)
Packaging, publishing, fetching, verifying, caching, and activating Runtime Packs across devices/nodes.

### Downgrade Prevention (kOA)
Rules preventing activation of an older (or policy-incompatible) Runtime Pack when it violates safety/compatibility constraints.

### Exchange (Kristal Exchange) (Kristal)
Canonical, content-addressed truth artifact compiled after validation.  
See Kristal (pinned): `vendor/kristal/02-schemas/exchange-manifest.schema.json` and core spec.

### Fail-Closed (kOA)
If declared integrity material (hashes/signatures/refs) is missing/invalid/mismatched, the operation must stop (no best-effort).

### Federation (Kristal)
Multi-authority model for shard/federation manifests, acceptance rules, and trust roots.  
See Kristal (pinned): `vendor/kristal/02-schemas/exchange-federation-manifest.schema.json`.

### Hash / Content Hash (Kristal)
Canonical hash of signing target / content; used for IDs and verification.  
See Kristal (pinned): `vendor/kristal/01-core-spec/ids-canonicalization-hashing.md`.

### Integrity Material (kOA)
Hashes, signatures, key references, and authority metadata required to verify an artifact. Verification is fail-closed.

### Konnaxion (kOA)
Distribution and platform layer facet responsible for pack fetch/verify/cache/activate/rollback and offline-first delivery behavior.

### Mandate Bundle (kOA)
Pinned policy/mandate context required for operation; if absent or revoked, the system must not operate.

### No Compile on Fail (kOA)
Hard gate: if validation fails, compilation (Exchange/Runtime Pack) must not run.

### No New Facts Downstream (kOA)
Render/execution layers must not invent facts; they must trace statements back to validated lineage or omit/refuse deterministically.

### Orgo (kOA)
Control plane: orchestrates pipeline stages, enforces gating, records Build/Release records, and governs operational work (Cases/Tasks).

### Pack (Runtime Pack) (Kristal)
Derived offline-executable artifact compiled from Exchange; consumed by Konnaxion/runtime.  
See Kristal (pinned): `vendor/kristal/02-schemas/runtime-pack-manifest.schema.json`.

### Pack Index (kOA)
Signed/verified index describing available packs in a channel (e.g., latest/pinned/revoked), used by Konnaxion.

### Policy Selections (kOA)
The set of enabled policies/profiles used for a build/run; must be recorded for reproducibility and audit.

### Provenance (kOA)
Recorded lineage linking outputs to inputs, configs, policies, and gate results (Build Record + artifact manifests).

### Render Bundle (kOA)
Deterministic renderer output envelope with trace coverage and events.  
kOA-native definition lives in `docs/30-artifacts/` (do not treat as Kristal-normative unless explicitly adopted in `docs/40-integration/kristal-v4/`).

### Release Record (kOA)
Operational record tying a build’s outputs to distribution intent (channels/cohorts/timestamps/status), enabling traceability and rollback.

### Resolution (SenTient) (kOA integration)
Mapping surfaces to candidates (QIDs/PIDs) + literal normalization + explicit ambiguity preservation.  
Kristal defines the Resolved Claim-IR artifact schema.

### Resolved Claim-IR (Kristal)
Resolution output format consumed by validation/compile.  
See Kristal (pinned): `vendor/kristal/02-schemas/resolved-claim-ir.schema.json`.

### Rollback (kOA)
Deterministic restoration to a known-good active pack state following activation failure, policy violation, or operator action.

### Schema (kOA)
A machine-validated contract defining artifact structure. Kristal schemas remain authoritative for Kristal artifacts.

### Shard (Kristal)
Partitioned Exchange unit with its own manifest and acceptance rules.  
See Kristal (pinned): `vendor/kristal/02-schemas/exchange-shard-manifest.schema.json`.

### Signature / Signing Profile (Kristal)
Signature envelope and verification semantics (key_id, alg, payload_hash, etc.).  
See Kristal (pinned): `vendor/kristal/01-core-spec/signatures-trust.md`.

### SenTient (kOA)
Resolution engine responsible for producing Resolved Claim-IR from Claim-IR while preserving uncertainty and determinism.

### Task (Orgo Task) (kOA)
Canonical unit of operational work with lifecycle state; executable by downstream actors (e.g., SwarmCraft) under governance rules.

### Trace Map (Render) (kOA)
Mapping from rendered assertions to supporting validated lineage (claims, entities, evidence); used to enforce “no new facts.”

### Trust Roots (Kristal + kOA deployment)
Pinned roots used to verify signatures (authority registry / deployment policy).  
See Kristal (pinned): `vendor/kristal/02-schemas/authority-registry.schema.json`.

### Validation Report (Kristal)
Deterministic acceptance output from validation stage; compile must be blocked on failure.  
See Kristal (pinned): `vendor/kristal/02-schemas/validation-report.schema.json`.

### Versioning (kOA)
Rules for evolving schemas/policies without breaking verification, reproducibility, or activation safety (see compatibility policy).

---