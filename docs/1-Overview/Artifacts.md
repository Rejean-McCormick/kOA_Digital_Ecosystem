# Artifacts

Artifacts are the **typed payloads** that cross boundaries between kOA components (and between kOA and Kristal). They exist so the system can be **deterministic, auditable, and operable** without hidden shared state.

## Two artifact families

### 1) Kristal artifacts (external, normative)
Kristal defines the contracts for:
- Claim-IR / Resolved Claim-IR
- Validation Report
- Exchange (canonical truth)
- Runtime Pack (offline-executable pack) and manifests/signatures

kOA treats these as **opaque references** (IDs + manifest references) and does not restate their schemas here.

### 2) kOA-native artifacts (kOA-owned)
kOA defines artifacts needed to **operate, govern, distribute, and observe** the ecosystem. Examples:
- **Governance:** Orgo Case, Orgo Task (work + approvals + audit trail)
- **Pipeline records:** Build Record, Release Record (what ran, what passed, what was promoted)
- **Distribution/runtime state:** Konnaxion State, activation/rollback records, cache/index artifacts
- **Policy context:** Mandate Bundle (pinned policy/mandate context)

## What artifacts are for (why they exist)

Artifacts make the system:
- **Reproducible:** build/release decisions can be reconstructed from pinned references
- **Fail-closed:** gates and verification have explicit, recorded outcomes
- **Auditable:** operations leave an append-only trail of what happened and why
- **Composable:** integrations exchange references, not implicit state

## How artifacts are handled (high-level rules)

### References, not payloads
- Prefer **content-addressed references** where possible.
- Do not embed large payloads inside operational artifacts; store pointers/hashes instead.
- References should include enough context to resolve the artifact (type + ID + hash/ref).

### Opaque IDs across boundaries
- Kristal artifact IDs and internals are treated as **opaque** by kOA-native records.
- kOA records store “what” (refs + outcomes), not Kristal’s internal structure.

## Contract expectations (what “done” looks like)

Each **kOA-native** artifact should have:
- A human-readable contract page (this wiki)
- A machine-validated schema (in-repo schemas folder)
- Clear versioning/compatibility expectations

## Minimal artifact lifecycle (operator view)

1. **Produced** by a component (e.g., ingest snapshot, build record, runtime state)
2. **Referenced** by governance/work records (Case/Task) and pipeline records (Build/Release)
3. **Promoted** via release controls (channel/cohort/pinning)
4. **Activated** by Konnaxion after verification (atomic switch, rollback-safe)
5. **Superseded** by a newer build/release or rolled back to last-known-good
6. **Audited** via append-only records and stored references

## Where to go next

- For the list of kOA-native artifacts: see `Artifacts-Operational.md`
- For Kristal artifacts and authoritative contracts: see `Integration-Kristal-v4.md`
- For operational behavior (release/rollback): see `Operations.md`