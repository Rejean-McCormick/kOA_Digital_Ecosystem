# Kristal v4 — Pinned Dependency

## Purpose

This section defines how the kOA Digital Ecosystem **depends on Kristal v4** without duplicating Kristal’s normative contracts (schemas, canonicalization rules, signing targets, and artifact definitions).

kOA MUST treat Kristal as an **external normative source** for Kristal-owned artifacts. Local copies of Kristal schemas or “illustrative manifests” are not permitted unless they are vendored verbatim from the pinned version and clearly labeled as such.

## Pin policy (normative)

kOA MUST pin Kristal v4 by **one** of these mechanisms:

1. **Git submodule** (preferred)
2. **Vendored subtree** at an immutable commit hash
3. **Artifact registry** download pinned by digest (if your build system supports it)

A floating reference (e.g., `main`, `latest`, unpinned tags) is not allowed.

## What is pinned

The pin MUST include:

- Kristal specification docs (the normative text)
- Kristal schemas (the executable contracts)
- Conformance tests (if provided) or the schema set necessary to run kOA conformance tests

Recommended repo layout (example):

- `vendor/kristal/`  (submodule or subtree)
  - `02-schemas/`
  - `03-artifacts/` (if present)
  - `04-protocols/` (if present)
  - `CHANGELOG.md` / `VERSION` / release notes (if present)

## Required metadata in kOA

kOA MUST record the Kristal pin in:

- `docs/40-integration/kristal-v4/index.md` (human-readable)
- `docs/40-integration/kristal-v4/contract-pointers.md` (normative links)
- Build/Release records (machine-verifiable)

At minimum, record:

- `kristal_version` (semantic version or release label)
- `kristal_commit` (full commit hash) or `kristal_digest` (registry digest)
- `kristal_schema_set_id` (optional: hash of the schema directory contents)

## Contract references

All kOA documentation MUST reference Kristal-owned artifacts via the pinned path, for example:

- `vendor/kristal/02-schemas/exchange-manifest.schema.json`
- `vendor/kristal/02-schemas/runtime-pack-manifest.schema.json`

kOA docs must not restate field lists from these schemas.

## Upgrade policy

Upgrading the Kristal pin MUST:

- be done as a single explicit change (one PR/change set)
- include a compatibility review against kOA’s conformance checks
- update `docs/40-integration/kristal-v4/legacy-compat.md` if any behavior changes are required
- be recorded in an ADR under `docs/90-reference/adr/`

## Enforcement (recommended)

kOA SHOULD add a CI check that fails if:

- Kristal schemas are duplicated under `docs/` outside the `vendor/kristal/` pin
- any kOA doc includes a Kristal manifest “example” that does not match the pinned schema set
- unpinned Kristal references appear in build scripts or docs