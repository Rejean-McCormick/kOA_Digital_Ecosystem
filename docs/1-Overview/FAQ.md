# FAQ

## What is this wiki?
This wiki explains the **kOA Digital Ecosystem**: what it does, its major components, and how it is operated in production.

## What is *not* in this wiki?
This wiki does **not** re-specify **Kristal** artifact contracts (schemas, canonicalization rules, signature formats, etc.). Kristal v4 remains the external source of truth for those contracts.

## Where is the Kristal spec?
See the pinned Kristal v4 reference and pointers in:
- [Integration: Kristal v4](Integration-Kristal-v4)

## What is “canonical” vs “informative” here?
- **Canonical for kOA**: ecosystem invariants, operational rules, component responsibilities, and **kOA-native** operational artifacts.
- **External canonical**: Kristal artifacts and schemas (Exchange, Runtime Pack, Validation Report, Claim-IR, Resolved Claim-IR, etc.).

## Which artifacts are “kOA-native”?
Artifacts owned by kOA components, for example:
- Orgo operational artifacts (Cases, Tasks, Build/Release Records)
- Konnaxion operational state and rollout/activation records (where standardized here)

Kristal artifacts remain externally specified.

## How do I know what schemas to validate against?
- For **kOA-native artifacts**, validate against the schemas shipped with this repo (operational artifact schemas).
- For **Kristal artifacts**, validate against the **pinned Kristal v4 schemas** referenced from the Kristal integration section.

## Why is there a hard “no redundancy” rule?
Duplicating normative contracts causes drift. kOA documentation stays stable by pointing to Kristal as the source of truth, and only documenting **kOA’s integration constraints, gates, and operational policies**.

## Where do I start if I’m implementing?
Start with:
- [Home](Home)
- [Principles & invariants](Principles-and-invariants)
- [Lifecycle](Lifecycle)
- [Components](Components)
- [Artifacts](Artifacts)
- [Integration: Kristal v4](Integration-Kristal-v4)
- [Operations](Operations)

## Where do I start if I’m operating?
Start with:
- [Operations](Operations)
- [Operations: Releases](Operations-Releases)
- [Operations: Rollbacks](Operations-Rollbacks)
- [Operations: Observability](Operations-Observability)
- [Operations: Incident response](Operations-Incident-response)

## How do changes get made safely?
- Changes to **kOA** invariants/interfaces/operational contracts should be accompanied by an ADR and conformance updates.
- Changes to **Kristal** contracts must happen in the pinned Kristal source of truth, and then kOA updates its Kristal integration profile/conformance accordingly.

## What should I do if I find a mismatch between kOA and Kristal?
Treat it as a **kOA integration issue** unless you are also changing the pinned Kristal version. Update:
- [Integration: Kristal v4](Integration-Kristal-v4)
- Any impacted operations or artifact pages that reference the integration behavior

## Does kOA require online connectivity to use Kristal artifacts?
No by default. Distribution/activation and runtime use are designed to be compatible with offline-first operation; connectivity is a deployment choice, not a contract requirement.

## What are the core components?
- **Orgo**: workflow/control plane + gating
- **SenTient**: resolution/reconciliation
- **Kristal**: truth pivot + compilation (externally specified)
- **Konnaxion**: distribution/activation + rollback safety
- **Architect**: deterministic rendering (no new facts)
- **SwarmCraft**: governed execution (optional)