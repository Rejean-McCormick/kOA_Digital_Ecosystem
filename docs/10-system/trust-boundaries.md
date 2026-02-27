# Trust Boundaries

**File:** `docs/10-system/trust-boundaries.md`  
**Status:** Normative (kOA)  
**External normative references:** Kristal v4 docs + schemas (pinned dependency)

---

## 1) Purpose

Define the hard boundaries that keep the ecosystem:
- auditable,
- offline-correct,
- tenant-safe,
- fail-closed when integrity is declared,
- resistant to downgrade/substitution and cross-tenant trust confusion.

This page is *system-level*. It does **not** restate Kristal artifact schemas.

---

## 2) Boundary map (what is allowed to change, where)

### 2.1 Truth boundary (canon boundary)
**Where canon is created:** deterministic validation → compilation into Kristal Exchange.  
**Downstream rule:** nothing downstream mutates canon; changes become new governed work that yields new artifacts.

### 2.2 Integrity boundary (verification boundary)
**Where bytes become trusted:** at publish (Orgo) and at activation/use (Konnaxion).  
If integrity material is declared, verification is mandatory and fail-closed.

### 2.3 Distribution boundary (activation boundary)
**Where “what runs locally” is decided:** Konnaxion activation pipeline (atomic switch, compatibility checks, downgrade prevention, rollback constraints).

### 2.4 Rendering boundary (no-new-facts boundary)
**Where language/UI is produced:** Architect-Render must not introduce unsupported facts; it must trace to canon.

### 2.5 Execution boundary (side-effects boundary)
**Where real-world/side-effect work happens:** SwarmCraft (or equivalent). Execution produces telemetry and artifacts, not canon edits.

### 2.6 Tenancy boundary (isolation boundary)
**Where data and trust must not cross:** tenant/environment boundaries across storage, trust roots, channels, and policy.

---

## 3) Trust roots and trust material

### 3.1 Trust roots are pinned and tenant-scoped
- Trust roots MUST be pinned per channel and scoped to tenant/environment.
- Correctness MUST NOT depend on fetching trust roots over the network at activation time.

### 3.2 Trust material types (typical)
- channel index signatures (optional but recommended)
- Runtime Pack Manifest signatures
- per-file inventory hashes in pack bundles
- authority registries / revocation lists (if your deployment uses them)

---

## 4) Orgo boundaries (control plane)

### 4.1 Gate enforcement
Orgo enforces stage ordering and gates (especially “no compile on fail”) and blocks publish if required integrity checks fail.

### 4.2 Evidence recording
Orgo records:
- build/release identifiers and artifact identifiers,
- signer identity (`key_id`) and signature metadata,
- policy selection / configuration identity,
- references needed for reproducibility and audit.

### 4.3 Tenant-safe trust handling
Orgo MUST not mix trust roots across tenants/channels and should manage rotation and revocation publication.

---

## 5) Konnaxion boundaries (distribution facet)

### 5.1 Activation rule (atomic and gated)
Konnaxion activates a pack only when:
1) integrity verification passes (as declared; fail-closed)  
2) manifest parses + schema validates  
3) referenced files exist  
4) runtime compatibility checks pass  
5) activation is an atomic switch (no partial activation)

### 5.2 Verification ordering (recommended)
1) verify channel index (if present)  
2) verify Runtime Pack Manifest signature(s)  
3) verify bundle file hashes (inventory)  
4) activate

### 5.3 Downgrade prevention + substitution safety
- Maintain deterministic, persisted state (per channel) for highest activated release.
- Reject same-release substitutions unless explicitly authorized.
- Do not activate revoked artifacts when a verified revocation mechanism exists.

### 5.4 Rollback is explicit and policy-authorized
Rollback must be deterministic given the same verified inputs and triggers, and must not occur silently.

---

## 6) Multi-tenancy boundaries

### 6.1 Data isolation
- Inputs, build artifacts, packs, logs, and telemetry are tenant-scoped.
- Access control and storage layout must prevent cross-tenant reads/writes.

### 6.2 Trust isolation
- Trust roots are tenant/environment/channel scoped.
- Signature verification must be tenant-correct (no cross-tenant key acceptance unless explicitly configured and auditable).

### 6.3 Policy isolation
- Mandates/policies and blueprint/config pins are tenant-scoped unless explicitly shared by design (and then treated as shared dependencies with explicit governance).

---

## 7) Failure modes (must be handled deterministically)

- **Wrong trust root set** (cross-tenant confusion) → hard fail (no activation/publish).
- **Tampered bytes** (hash/signature mismatch) → fail-closed.
- **Downgrade attempt** (lower release_id without rollback authorization) → refuse activation.
- **Substitution** (same release_id, different artifact_id) → refuse unless reissue authorization exists.
- **Revoked pack/key** → refuse activation when revocation is enforced.

---

## 8) Minimum conformance checklist

- [ ] Truth boundary enforced (no compile on fail; no downstream canon edits).
- [ ] Pinned trust roots per channel; offline verification possible.
- [ ] Fail-closed on any declared integrity material.
- [ ] Atomic activation; deterministic rollback behavior.
- [ ] Downgrade prevention + substitution safety.
- [ ] Tenant isolation for data + trust roots + policy.
- [ ] Orgo records build/release provenance and integrity metadata (`key_id`, signatures, hashes).