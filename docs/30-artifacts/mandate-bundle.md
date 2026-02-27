# Mandate Bundle

**Owner:** kOA (Keter / Mandate)
**Role:** A pinned, versioned bundle of governance inputs that shape what the ecosystem is allowed to do and how it must behave.

The Mandate Bundle is **kOA-owned**. It is not a Kristal artifact. It provides policy and constraints that upstream and control-plane components use to decide what work is permitted and how it must be processed.

## Purpose

* Provide a single, content-addressed “governance package” that can be referenced by builds, releases, and audits.
* Enable reproducible decisions: given the same Mandate Bundle, policy evaluation results should be consistent.
* Separate governance configuration from runtime execution and from canonical truth artifacts.

## What it contains

A Mandate Bundle typically includes:

* **Policy selections**: identifiers of active policies (with versions).
* **Validation policy config**: which validators run, thresholds, required checks.
* **Distribution policy config**: channel rules, pinning rules, downgrade prevention, signer requirements.
* **Rendering policy config**: determinism constraints, refusal/omission rules, trace coverage requirements.
* **Execution policy config**: allowed task categories, rate limits, escalation rules.
* **Key/trust policy**: trusted signer sets, revocations, key-rotation rules.
* **Metadata**: bundle ID, created time, author/approver identity, change log reference.

## Consumers

* **Orgo:** uses it to enforce gate decisions and stage ordering, and to record policy provenance in Build/Release records.
* **Validation stage:** uses it to decide which checks are required and how results are scored.
* **Konnaxion:** uses it to enforce verification/activation rules and rollback constraints.
* **Architect-Render:** uses it to enforce no-new-facts and trace/omission/refusal policies.
* **SwarmCraft:** uses it to enforce execution constraints.

## Invariants

* **Content-addressed:** bundle ID is derived from canonical bytes of the bundle.
* **Immutable:** a published bundle is never edited; changes create a new bundle.
* **Auditable:** includes provenance (who approved it, when, why).
* **Deterministic evaluation:** policy interpretation must be stable under the pinned policy engine/version.
* **Explicit scope:** bundle clearly states which parts apply to which stages (validation vs distribution vs rendering vs execution).

## Failure modes

* Missing or ambiguous policy identifiers
* Non-deterministic policy evaluation (engine drift)
* Unauthorized bundle publication (approval chain broken)
* Incompatible policy set (e.g., requires validators not available)

## Storage and referencing

* Bundles should be stored in an artifact store with:

  * content hash
  * human-readable version tag (optional)
  * approval metadata
* Build/Release records reference the Mandate Bundle by content-addressed ID.

## Schema

* JSON schema: `docs/30-artifacts/schemas/mandate-bundle.schema.json`
