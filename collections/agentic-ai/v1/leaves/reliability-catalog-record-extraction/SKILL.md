---
name: reliability-catalog-record-extraction
description: Use when you must verify that a machine-readable companion catalog's
  record counts and category composition match a published claim (e.g., 206 records
  = 193 gated practices + 13 research leads).
when_to_use_negative:
- Input is unstructured prose or a paper PDF rather than the machine-readable companion
  catalog; the workflow depends on parseable records with ERCA-NNN fields
- You need to run or evaluate coding agents against reliability protocols (use the
  minimum reliability pass scaffold instead of catalog extraction)
- The catalog revision has changed and the published composition (206 records) is
  no longer the expected target without re-checking the versioned release notes
edam_operation: http://edamontology.org/operation_2409
tools:
- name: git
  role: Clone the deposited repository to obtain the companion catalog and SHA256SUMS
    files
- name: Python 3
  role: Load companion/catalog. with a Python module, enumerate records, validate
    ERCA-NNN identifiers, classify records, and write catalog_counts.csv
- name: sha256sum
  role: Confirm companion file integrity via sha256sum -c SHA256SUMS before trusting
    tallies
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:97f85fc1062253f6bf490bec63444452a2c6a22520a656592b212b3acde7b0fb
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Python 3
  - git
  - sha256sum
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# reliability-catalog-record-extraction

## Summary

Extract, validate, and tally records from the machine-readable companion catalog of the Engineering Reliable Coding Agents monograph to confirm the reported composition of 206 reliability records (193 gated practices, 56 developed in depth, plus 13 research leads).

## When to use

Use when you must verify that a machine-readable companion catalog's record counts and category composition match a published claim (e.g., 206 records = 193 gated practices + 13 research leads).

## When NOT to use

- Input is unstructured prose or a paper PDF rather than the machine-readable companion catalog; the workflow depends on parseable records with ERCA-NNN fields
- You need to run or evaluate coding agents against reliability protocols (use the minimum reliability pass scaffold instead of catalog extraction)
- The catalog revision has changed and the published composition (206 records) is no longer the expected target without re-checking the versioned release notes

## Inputs

- Deposited GitHub repository sjarmak/engineering-reliable-coding-agents (companion directory)
- companion/catalog. machine-readable catalog file keyed by ERCA-NNN identifiers
- companion/SHA256SUMS manifest for integrity verification

## Outputs

- catalog_counts.csv with per-category tallies and comparison verdicts against the reported composition
- List of records flagged for missing or malformed ERCA-NNN identifiers
- sha256sum -c verification status for companion files

## How to apply

Clone the deposited repository with git (https://github.com/sjarmak/engineering-reliable-coding-agents.git) to obtain the companion files, then load companion/catalog. with Python 3's module and enumerate all records. Extract and validate the stable ERCA-NNN identifier on every record, flagging any record missing a well-formed identifier. Classify each record using its practice-level fields into one of three categories: gated practice marked as developed (chapter-level treatment), gated practice appearing only in the companion, or research lead. Compare the tallies against the reported composition (206 total, 193 gated practices with 56 developed and 137 companion-only, 13 leads), writing tallies and comparison verdicts to catalog_counts.csv, and confirm the SHA256SUMS check for companion files passes with sha256sum -c.

## Related tools

- **git** (Clone the deposited repository to obtain the companion catalog and SHA256SUMS files)
- **Python 3** (Load companion/catalog. with a Python module, enumerate records, validate ERCA-NNN identifiers, classify records, and write catalog_counts.csv)
- **sha256sum** (Confirm companion file integrity via sha256sum -c SHA256SUMS before trusting tallies)

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && (cd engineering-reliable-coding-agents/companion && sha256sum -c SHA256SUMS)
```

## Evaluation signals

- Enumerated record count equals exactly 206 reliability records
- Category tallies resolve to 193 gated practices (56 developed in depth, 137 companion-only) plus 13 research leads
- Every catalog record carries a well-formed ERCA-NNN identifier with zero flagged records
- (cd companion && sha256sum -c SHA256SUMS) reports no checksum failures
- catalog_counts.csv verdicts show no discrepancy against the reported composition

## Limitations

- The original interactive searches did not preserve every machine-issued query in a publication-ready log, so the companion distinguishes retained records from reconstructions; counts must be interpreted with that distinction in mind
- The catalog is versioned, so a fixed composition like 206 records is only expected for the matching release; verify the SHA256SUMS check before treating tallies as a reproduction failure

## Evidence

- [results] The companion catalog indexes 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads, matching the reported composition.: [evidence span withheld — anchor status: not_found]
- [methods] Extract and validate the stable ERCA-NNN identifier on every catalog record, flagging any record missing a well-formed identifier.: [evidence span withheld — anchor status: not_found]
- [methods] Compare the tallies against the reported composition of 206 total records, 193 gated practices with 56 developed and 137 companion-only, and 13 leads, recording any discrepancy.: [evidence span withheld — anchor status: not_found]
- [abstract] The monograph contributes a versioned catalog of 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads.: [evidence span withheld — anchor status: not_found]
