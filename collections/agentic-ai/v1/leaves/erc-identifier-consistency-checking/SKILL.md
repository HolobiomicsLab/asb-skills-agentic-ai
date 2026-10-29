---
name: erc-identifier-consistency-checking
description: Use when auditing a machine-readable practice/evidence catalog for broken
  or missing record identifiers, or when verifying that a deposited companion catalog's
  record tallies match the composition claimed in the associated manuscript.
when_to_use_negative:
- The catalog has not been checksum-verified against SHA256SUMS — classification results
  would be unreliable if companion files differ from the deposit
- The dataset is a benchmark task set (e.g. CodeScaleBench csb-v1-mixed371) or mined
  agent-evaluation tasks (CodeProbe) rather than the ERCA evidence/practice catalog
  with ERCA-NNN identifiers
- You need to re-run reliability protocols (e.g. the six-step minimum reliability
  pass) rather than audit catalog record identity and counts
tools:
- name: git
  role: Clone the deposited engineering-reliable-coding-agents repository to obtain
    the companion catalog files for identifier validation
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: Python 3
  role: Load companion/catalog. with the  module, enumerate records, validate ERCA-NNN
    identifiers, classify records, and emit tallies
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: sha256sum
  role: Verify companion file integrity with sha256sum -c SHA256SUMS so identifier
    checks run against the exact deposited bytes
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:ec4698995dba828a49c03334cf567d181cd2ef4049ce91d71e7b1deec1863279
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ercа-identifier-consistency-checking

## Summary

Validates that every record in the Engineering Reliable Coding Agents (ERCA) companion catalog carries a well-formed, stable ERCA-NNN identifier, and cross-checks record classification and composition (206 reliability records: 193 gated practices with 56 developed in depth, plus 13 research leads) against the monograph's reported counts.

## When to use

Use when auditing a machine-readable practice/evidence catalog for broken or missing record identifiers, or when verifying that a deposited companion catalog's record tallies match the composition claimed in the associated manuscript.

## When NOT to use

- The catalog has not been checksum-verified against SHA256SUMS — classification results would be unreliable if companion files differ from the deposit
- The dataset is a benchmark task set (e.g. CodeScaleBench csb-v1-mixed371) or mined agent-evaluation tasks (CodeProbe) rather than the ERCA evidence/practice catalog with ERCA-NNN identifiers
- You need to re-run reliability protocols (e.g. the six-step minimum reliability pass) rather than audit catalog record identity and counts

## Inputs

- Cloned companion repository sjarmak/engineering-reliable-coding-agents (companion/catalog. machine-readable catalog file)
- companion/SHA256SUMS checksum manifest
- Reported composition from the monograph: 206 total records, 193 gated practices (56 developed in depth, 137 companion-only), 13 research leads

## Outputs

- catalog_counts.csv containing per-category tallies and comparison verdicts against the reported composition
- List of records flagged for missing or malformed ERCA-NNN identifiers
- sha256sum -c pass/fail confirmation for the companion files

## How to apply

Clone the deposited repository (github.com/sjarmak/engineering-reliable-coding-agents) with git so the companion files are checked out exactly as deposited. Load companion/catalog. with Python 3's  module and enumerate all records, extracting and validating the stable ERCA-NNN identifier on each record and flagging any record lacking a well-formed identifier. Classify each record — gated practice developed in depth (chapter-level treatment), gated practice appearing only in the companion, or research lead — using the record's practice-level fields. Compare tallies against the reported composition of 206 total records, 193 gated practices (56 developed, 137 companion-only), and 13 leads, recording discrepancies to catalog_counts.csv. Finally, run sha256sum -c against the companion's SHA256SUMS to confirm the checked-out files match the deposited checksums before trusting any tally.

## Related tools

- **git** (Clone the deposited engineering-reliable-coding-agents repository to obtain the companion catalog files for identifier validation) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **Python 3** (Load companion/catalog. with the  module, enumerate records, validate ERCA-NNN identifiers, classify records, and emit tallies) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **sha256sum** (Verify companion file integrity with sha256sum -c SHA256SUMS so identifier checks run against the exact deposited bytes) — https://github.com/sjarmak/engineering-reliable-coding-agents.git

## Examples

```
(cd companion && sha256sum -c SHA256SUMS)
```

## Evaluation signals

- Every catalog record yields a well-formed ERCA-NNN identifier; zero records are flagged as missing or malformed
- Total record count equals 206 reliability records
- Gated-practice tally equals 193, with 56 developed in depth and 137 companion-only, and research-lead tally equals 13 — matching the reported composition
- (cd companion && sha256sum -c SHA256SUMS) reports all companion files OK
- catalog_counts.csv is written with tallies and comparison verdicts for every category

## Limitations

- The companion distinguishes retained records from reconstructions because the original interactive searches did not preserve every machine-issued query in a publication-ready log, so some records may be reconstructions rather than retained originals
- The monograph's review is structured rather than exhaustive and evidence strength varies by topic, so record composition reflects the curation gates, not an exhaustive literature census
- Classification of developed vs. companion-only practices depends on practice-level fields in the companion records; field drift between the catalog and the manuscript can shift the 56-developed tally

## Evidence

- [methods] Load companion/catalog. with Python 3's module and enumerate all records.: [evidence span withheld — anchor status: not_found]
