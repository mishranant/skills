# Seed glossary (template)

Fast-path definitions for **stable, common** internal terms so the skill doesn't
re-look-up the obvious ones. This is a convenience cache, **not** authoritative.

How to use this file:
- Replace the example rows below with your own org's stable, frequently-seen
  terms. Keep only entries you're confident about.
- Mark lower-confidence entries **(verify)** — confirm via your glossary / docs
  before relying on them.
- Any term **not** listed here must be resolved live (organization glossary →
  service/component definitions → internal docs/wiki → chat → code search).
- If a live lookup contradicts an entry here, trust the live source and fix this
  file.

## Generic infra terms (safe defaults, not org-specific)

| Term | Expansion | Plain meaning |
|------|-----------|---------------|
| substrate | — | The underlying cloud/platform a service runs on (e.g. AWS, GCP, on-prem). |
| deployment cell / instance | — | One isolated running copy of a service, scoped to a region/environment. |
| add-on / module | — | A reusable infrastructure module that provisions one capability (DB, cache, secrets path, mesh, …). |
| structured config | — | Per-environment config (often YAML) that tunes a module without editing its code. |
| BOM | Bill of Materials | A pinned set of dependency/module versions promoted together. |
| RCA | Root Cause Analysis | The confirmed underlying cause of an incident. |
| change case / change request | — | A formal approval record required before a production change. |

## Your org's terms (fill in)

| Term | Expansion | Plain meaning |
|------|-----------|---------------|
| <ACRONYM> | <full expansion> | <one-line plain meaning> |
| <internal-tool-name> | — | <what it does, in plain engineering terms> |
| <work-tracker> | — | <the internal planning/ticketing system; ticket id format> |
| <ci-system> | — | <internal CI system; where the pipeline is defined> |
| <metrics-system> | — | <internal time-series/monitoring system> |
| <logs-system> | — | <internal log search/aggregation platform> |
