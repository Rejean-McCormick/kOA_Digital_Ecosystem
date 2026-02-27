# FAQ

## What is this documentation set?
This is the **kOA Digital Ecosystem** documentation: how the ecosystem is structured, how its components behave, and how it operates in production.

## What is *not* in this documentation set?
It does **not** re-specify **Kristal** artifact contracts (schemas, canonicalization rules, signature formats, etc.). Those are owned by the **Kristal v4** documentation and schemas.

## Where is the Kristal spec?
See: `docs/40-integration/kristal-v4/` (especially `pinned-dependency.md` and `contract-pointers.md`). This is the single place in the kOA docs that points to the pinned Kristal v4 source-of-truth.

## What is “canonical” vs “informative” here?
- **Canonical for kOA**: ecosystem invariants, operational rules, node responsibilities, and kOA-native artifact contracts (e.g., Orgo Case/Task, Build/Release records).
- **External canonical**: Kristal artifacts and schemas (Exchange, Runtime Pack, Validation Report, etc.).

## Which artifacts are “kOA-native”?
Artifacts owned by kOA components (examples):
- Orgo operational artifacts (Cases, Tasks, Build/Release records)
- Konnaxion operational state/telemetry envelopes (if defined here)
Kristal artifacts remain externally specified.

## How do I know what schemas to validate against?
- For **kOA-native artifacts**, validate against schemas in `docs/30-artifacts/schemas/`.
- For **Kristal artifacts**, validate against the pinned Kristal v4 schemas referenced in `docs/40-integration/kristal-v4/contract-pointers.md`.

## Why is there a hard “no redundancy” rule?
Duplicating normative contracts causes drift. kOA docs stay stable by pointing to Kristal v4 as the source-of-truth, and only documenting kOA’s integration constraints and operational policies.

## Where do I start if I’m implementing?
- `docs/00-overview/system-at-a-glance.md`
- `docs/10-system/architecture.md`
- `docs/60-guides/implementers.md`
- `docs/40-integration/kristal-v4/conformance.md`

## Where do I start if I’m operating?
- `docs/50-operations/pipeline.md`
- `docs/50-operations/releases.md`
- `docs/50-operations/rollback.md`
- `docs/50-operations/incident-response.md`

## How do changes get made safely?
- Changes to **kOA** invariants/interfaces: add/update an ADR under `docs/90-reference/adr/`.
- Changes to **Kristal** contracts: must happen in the pinned Kristal v4 source; then kOA updates its integration profile/conformance docs accordingly.

## What should I do if I find a mismatch between kOA and Kristal?
Treat it as a **kOA integration bug** (not a Kristal contract bug) unless you are also changing the pinned Kristal version. Update:
- `docs/40-integration/kristal-v4/koa-profile.md` (policy/constraints)
- `docs/40-integration/kristal-v4/conformance.md` (tests/acceptance)
- Any impacted kOA ops/guides that reference the integration behavior

## Does kOA require online connectivity to use Kristal artifacts?
No by default. Distribution/activation and runtime use should be compatible with offline-first operation; connectivity is a deployment choice, not a contract requirement.

## What are the core components?
- **Orgo**: workflow/control plane + gating
- **SenTient**: resolution/reconciliation
- **Kristal**: truth pivot + compilation (externally specified)
- **Konnaxion**: distribution/activation + rollback safety
- **Architect**: deterministic rendering (no new facts)
- **SwarmCraft**: execution (optional)
- **EkoH**: ledger/trust mechanisms (if deployed)