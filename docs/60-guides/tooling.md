# Tooling

**Purpose:** Describe the tooling expectations for working with kOA docs, artifacts, builds, releases, and conformance—without embedding Kristal contract definitions.

## 1) Documentation toolchain (recommended)

* **Markdown-first** docs under `docs/`
* A static site generator (pick one):

  * MkDocs (simple)
  * Docusaurus (richer)
  * Sphinx/MD (if you already use it)

Baseline expectations:

* Link checking in CI (no broken internal links)
* Spellcheck / style lint (optional)
* Build the docs site on every main-branch merge

## 2) Schema and contract tooling

kOA-owned schemas live under:

* `docs/30-artifacts/schemas/`

Recommended tooling:

* JSON Schema validation in CI (Draft 2020-12)
* Schema versioning policy:

  * bump `urn:koa:schema:<name>:<major>` when breaking
  * semver `record_version` inside records (e.g., `1.0.0`)

Kristal schemas:

* Must be referenced from the pinned Kristal v4 dependency (do not copy/paste into kOA).

## 3) Build tooling (Orgo)

Minimum CLI/automation capabilities:

* `orgo build start` (create build + record trigger)
* `orgo build status <build_id>`
* `orgo build artifacts <build_id>` (list content refs)
* `orgo build export <build_id>` (export Build Record + referenced metadata)

Recommended:

* Determinism checks (same inputs/policy/toolchain → same content refs)
* Artifact store client tooling (push/pull by content ref)

## 4) Release tooling

Minimum CLI/automation capabilities:

* `orgo release create --build <id> --channel <name> --cohort <selector>`
* `orgo release promote --release <id> --percent <n>`
* `orgo release pin --release <id> --pack <ref>`
* `orgo release rollback --release <id> [--policy pinned|lkg]`
* `orgo release status <id>`

Konnaxion-side tools (often separate):

* `konnaxion verify <pack_ref>`
* `konnaxion activate <pack_ref>`
* `konnaxion rollback [--to <pack_ref>|--policy lkg]`
* `konnaxion active`

## 5) Conformance and validation tooling

kOA should maintain a conformance suite that checks:

* Build Record schema validity
* Release Record schema validity
* Konnaxion activation invariants (fail-closed, atomic, deterministic rollback)
* Kristal artifact conformance via pinned Kristal schemas:

  * Exchange/Pack artifacts validate against the pinned spec
  * canonicalization identifiers match the pinned requirements
  * signatures/keys policy enforced (as configured in kOA profile)

## 6) Operational observability tooling

Minimum:

* Structured logs for build/release/activation events
* Metrics export:

  * build stage durations
  * validation pass/fail
  * activation success/fail
  * rollback events and triggers
* Trace/audit store for immutable records

Recommended:

* Dashboard templates per channel/cohort
* Alert rules tied to rollback triggers

## 7) Repo maintenance tooling (quality gates)

* Pre-commit hooks (format + lint)
* CI gates:

  * docs build
  * link check
  * schema validation
  * conformance tests
  * security scanning for dependencies (as applicable)
