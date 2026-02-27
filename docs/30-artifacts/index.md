# kOA Artifacts

**Scope:** kOA-owned artifacts only (operational + distribution + governance).  
**Non-goal:** This section does not redefine Kristal artifacts (Claim-IR, Resolved Claim-IR, Validation Report, Exchange, Runtime Pack). Those are normatively defined in Kristal v4 and referenced from `docs/40-integration/kristal-v4/`.

---

## What belongs here

kOA artifacts are typed payloads that exist to operate, govern, distribute, and observe the ecosystem. Examples include:

- **Orgo governance artifacts:** Case, Task, routing labels, audit objects
- **Pipeline operational records:** Build Record, Release Record
- **Distribution/runtime state:** Konnaxion State, Activation Record, Rollback Record, Cache Indexes
- **Mandate/policy bundles (kOA deployment):** Mandate Bundle (pinned mandate/policy context used by Orgo/Konnaxion)

Each artifact has:
- A **documented contract** (human-readable)
- A **JSON Schema** (machine-validated) under `docs/30-artifacts/schemas/`
- **Versioning/compat** expectations (see `docs/10-system/determinism.md` and `docs/40-integration/kristal-v4/koa-profile.md`)

---

## Artifact index

### Governance (Orgo)
- `orgo-case.md`
- `orgo-task.md`

### Pipeline records
- `build-record.md`
- `release-record.md`

### Policy / mandate
- `mandate-bundle.md`

### Distribution/runtime
- `konnaxion-state.md`

---

## Schemas

Schemas live in:
- `docs/30-artifacts/schemas/`

Naming rules:
- One schema per artifact
- `$id` must be stable and owned by kOA (do not use Kristal namespaces)
- `additionalProperties: false` unless explicitly justified
- Use semver for schema versioning inside the artifact where needed

---

## References

- Kristal v4 integration: `docs/40-integration/kristal-v4/`
- System invariants: `docs/10-system/trust-boundaries.md`, `docs/10-system/determinism.md`