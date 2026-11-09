---
name: dblp-publication-adjudication-analysis
description: Use when a review audit provides an author-vs-venue adjudication CSV
  from DBLP and you must tally unique publications, populated cells, and works missing
  from prior evidence sets.
when_to_use_negative:
- The audit artifact is a screening-decisions or search-log CSV (screening outcomes
  38/11/1/27 or retrieval coverage figures), not an author-adjudication table — those
  have their own reconciliation steps.
- No DBLP adjudication CSV exists and publications cannot be resolved to a resolvable
  scholarly identity, so cell-level tabulation is not possible.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3068
- http://edamontology.org/topic_0091
tools:
- name: Python 3
  role: csv/standard-library module used to load and aggregate the dblp-author-adjudication-2026-08.csv
    cells into the publication tally
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: git
  role: clone the companion repository to locate the adjudication CSV and retained-record
    files under companion/
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:c5b6ea189505211735491645c243d20f37b33e788ed22ef4f0ed02c4bcf8317a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3068
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# dblp-publication-adjudication-analysis

## Summary

Aggregate a DBLP author-adjudication CSV (e.g. dblp-author-adjudication-2026-08.csv) into a cell-and-publication tally used in a multivocal literature-review update audit, reporting unique publications, per-cell coverage, and records absent from prior evidence sets.

## When to use

Use when a review audit provides an author-vs-venue adjudication CSV from DBLP and you must tally unique publications, populated cells, and works missing from prior evidence sets.

## When NOT to use

- The audit artifact is a screening-decisions or search-log CSV (screening outcomes 38/11/1/27 or retrieval coverage figures), not an author-adjudication table — those have their own reconciliation steps.
- No DBLP adjudication CSV exists and publications cannot be resolved to a resolvable scholarly identity, so cell-level tabulation is not possible.

## Inputs

- dblp-author-adjudication-2026-08.csv (DBLP author adjudication table)
- prior evidence-set record lists from the companion repository (companion/ retained-record files)
- evidence-arithmetic-summary.csv schema (figure name, value, source file)

## Outputs

- DBLP tally: unique publications, total cells, and count absent from prior sets (55 / 64 / 50)
- evidence-arithmetic-summary.csv row(s) citing dblp-author-adjudication-2026-08.csv as source

## How to apply

Load the adjudication CSV (dblp-author-adjudication-2026-08.csv) with Python 3's standard-library csv module and treat each cell as an author-venue publication slot. Aggregate across cells to count unique publications (the reported tally is 55 unique publications across 64 cells) and flag which publications are absent from prior evidence sets (50 of 55 were absent). Cross-check cell counts so that per-cell totals reconcile to the unique-publication count, and emit the result as one row per figure in an evidence-arithmetic-summary.csv (figure name, value, source file) alongside the other audit figures (164 scholarly works, 206 catalog records). Validation requires every tabulated value to equal the reported figure and the output CSV to parse with all expected fields present.

## Related tools

- **Python 3** (csv/standard-library module used to load and aggregate the dblp-author-adjudication-2026-08.csv cells into the publication tally) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **git** (clone the companion repository to locate the adjudication CSV and retained-record files under companion/) — https://github.com/sjarmak/engineering-reliable-coding-agents.git

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && python3 -c "import csv,collections; rows=list(csv.DictReader(open('companion/dblp-author-adjudication-2026-08.csv'))); pubs={r['publication_id'] for r in rows}; print(len(pubs), len(rows))"
```

## Evaluation signals

- Unique-publication count equals 55 and total cell count equals 64 for dblp-author-adjudication-2026-08.csv
- Absent-from-prior-sets count equals 50 of the 55 unique publications
- Per-cell assignments reconcile: no publication is double-counted across cells and cell total (64) decomposes consistently into unique works (55)
- evidence-arithmetic-summary.csv parses with all expected fields (figure name, value, source file) and its DBLP row cites dblp-author-adjudication-2026-08.csv
- Catalog identity cross-check holds: 206 = 193 gated practices + 13 research leads

## Limitations

- The tally reflects a bounded update audit window (Code Intelligence Digest editions July 27 - August 5); figures are edition-scoped, not a comprehensive DBLP census.
- ACM Digital Library, IEEE Xplore, and Scopus searches were not performed and are reported as a disclosed source limitation, so absence from prior sets may partly reflect retrieval gaps.
- Original interactive searches did not preserve every machine-issued query in a publication-ready log; the companion distinguishes retained records from reconstructions.

## Evidence

- [other] Aggregate dblp-author-adjudication-2026-08.csv into the DBLP tally of 55 unique publications across 64 cells with 50 absent from prior sets.: [evidence span withheld — anchor status: not_found]
- [other] Load CSVs with Python stdlib: [evidence span withheld — anchor status: not_found]
- [other] Cross-check catalog identity: [evidence span withheld — anchor status: not_found]
