---
name: catalog-record-count-verification
description: Use when a deposited companion repository claims a specific catalog composition
  (e.g. 206 records = 193 gated practices + 13 research leads) that must be independently
  reproduced from the catalog file before citing or extending it.
when_to_use_negative:
- The goal is screening or retrieval-coverage arithmetic (search-log.csv, screening-decisions.csv,
  publisher-coverage-status.) rather than catalog record composition — a separate
  audit skill covers those figures
- The catalog lacks stable per-record identifiers, so no ERCA-NNN-style identity validation
  can be performed and classification cannot be anchored to a record ID
- Records must be re-derived from raw sources rather than verified against a deposited
  catalog snapshot; the checksum manifest would fail and the tallies would not trace
  to the reported edition
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_4015
tools:
- name: git
  role: clone the deposited companion repository to obtain companion/catalog. and
    SHA256SUMS before any counting
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: Python 3
  role: load and enumerate catalog records, validate ERCA-NNN identifiers, classify
    records by practice-level fields, and emit catalog_counts.csv
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: sha256sum
  role: verify companion file integrity (sha256sum -c SHA256SUMS) so verified counts
    are attributable to unmodified deposited files
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_content_hash: sha256:75433b5ddc110cafb18ed05dcca748ee0bcf9e93e6efc6d4387c4fef16c1cc96
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents.git
  tools:
  - Python
  - Python 3
  - git
  - sha256sum
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_4015
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# catalog-record-count-verification

## Summary

Reproduce and verify the record composition of a machine-readable practice catalog by enumerating every record, validating its stable identifier, and classifying it into the article-reported categories (gated practices vs research leads) to confirm the deposited counts match the reported tallies.

## When to use

Use when a deposited companion repository claims a specific catalog composition (e.g. 206 records = 193 gated practices + 13 research leads) that must be independently reproduced from the catalog file before citing or extending it.

## When NOT to use

- The goal is screening or retrieval-coverage arithmetic (search-log.csv, screening-decisions.csv, publisher-coverage-status.) rather than catalog record composition — a separate audit skill covers those figures
- The catalog lacks stable per-record identifiers, so no ERCA-NNN-style identity validation can be performed and classification cannot be anchored to a record ID
- Records must be re-derived from raw sources rather than verified against a deposited catalog snapshot; the checksum manifest would fail and the tallies would not trace to the reported edition

## Inputs

- companion/catalog. (machine-readable catalog file from the cloned repository, one record per reliability entry)
- companion/SHA256SUMS (checksum manifest for companion files)
- reported composition claim (e.g. 206 total, 193 gated practices with 56 developed, 13 research leads)

## Outputs

- catalog_counts.csv with per-category tallies and comparison verdicts against the reported composition
- list of records with missing or malformed ERCA-NNN identifiers
- SHA256SUMS verification result (pass/fail) for companion files

## How to apply

Clone the companion repository with git (https://github.com/sjarmak/engineering-reliable-coding-agents.git) and load companion/catalog. with Python 3 to enumerate all records. Extract and validate the stable ERCA-NNN identifier on every record, flagging any record missing a well-formed identifier. Classify each record using its practice-level fields into one of: gated practice marked as developed (chapter-level treatment), gated practice appearing only in the companion, or research lead; then compare the tallies against the reported composition of 206 total records, 193 gated practices (56 developed in depth, 137 companion-only), and 13 research leads, recording any discrepancy. Cross-check the identity 206 = 193 + 13 as a constraint on the classification, and write the tallies and verdicts to catalog_counts.csv. Finally confirm companion file integrity with sha256sum -c SHA256SUMS so the verified counts trace to unmodified companion files.

## Related tools

- **git** (clone the deposited companion repository to obtain companion/catalog. and SHA256SUMS before any counting) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **Python 3** (load and enumerate catalog records, validate ERCA-NNN identifiers, classify records by practice-level fields, and emit catalog_counts.csv) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **sha256sum** (verify companion file integrity (sha256sum -c SHA256SUMS) so verified counts are attributable to unmodified deposited files) — https://github.com/sjarmak/engineering-reliable-coding-agents.git

## Examples

```
python3 scripts/check_release_counts.py && (cd companion && sha256sum -c SHA256SUMS)
```

## Evaluation signals

- Enumerated record count equals the reported catalog size: 206 reliability records
- Category split reproduces exactly 193 gated practices + 13 research leads, with zero unclassified records
- Every record carries a well-formed ERCA-NNN identifier; no missing or malformed IDs are flagged
- Developed-practice tally equals the reported 56, with the remaining 137 gated practices classified as companion-only
- Identity constraint holds: 206 = 193 + 13, and sha256sum -c SHA256SUMS passes for all companion files

## Limitations

- The verification confirms the deposited catalog matches the reported composition; it cannot establish that the underlying evidence supports each record's admission (the scholarly/practitioner independence gate lives in the review workflow, not the catalog file)
- The review is structured rather than exhaustive and evidence strength varies by topic, so a passing count check does not certify per-record evidence quality
- Record classification depends on the catalog's own practice-level fields; if those fields are mislabeled upstream, tallies will reproduce a reported figure without the classification being correct

## Evidence

- [abstract] The monograph contributes a versioned catalog of 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads: "The monograph contributes a versioned catalog of 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads"
- [supplementary] workflow: classify records and validate identifiers: [evidence span withheld — anchor status: not_found]
- [methods] workflow: checksum verification: "(cd companion && sha256sum -c SHA256SUMS)"
- [methods] workflow: clone and load catalog: "git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git"
- [supplementary] findings: reported composition matches: [evidence span withheld — anchor status: not_found]
- [abstract] scope statement: structured review: "The review is structured rather than exhaustive, evidence strength varies by topic, and empirical results remain dependent on workload and system configuration"
