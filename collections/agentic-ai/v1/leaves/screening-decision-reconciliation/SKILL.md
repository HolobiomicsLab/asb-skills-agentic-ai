---
name: screening-decision-reconciliation
description: Use when auditing a review edition's reported corpus and screening counts
  (e.g. 38 screened / 11 admitted / 27 deferred) against companion repository artifacts
  to verify each published figure.
when_to_use_negative:
- The original interactive searches did not preserve every machine-issued query in
  a publication-ready log, so this skill cannot re-derive raw retrieval sessions —
  the companion distinguishes retained records from reconstructions
- Input is a live literature search rather than a frozen companion snapshot; the figures
  apply to the bounded update audit of Code Intelligence Digest editions (July 27
  - August 5), not to new searching
- 'You need venue coverage beyond the artifact set: ACM Digital Library, IEEE Xplore,
  and Scopus searches were not performed and are a disclosed source limitation, not
  zero-result evidence'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: git
  role: Clone the companion repository to obtain the retained-record files under companion/
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: Python 3
  role: Parse the CSV/JSON companion artifacts with the standard-library csv/ modules
    and write evidence-arithmetic-summary.csv
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:afdcebd0ad92101f19212c6fd2eb4dfddc1586ddbfde9e2977d810b384a81181
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# screening-decision-reconciliation

## Summary

Reproduce and cross-check the source-assembly, screening, and venue-coverage arithmetic of a multivocal review by tabulating companion CSV/JSON artifacts (search-log.csv, screening-decisions.csv, source-snapshot, publisher-coverage-status) and writing a per-figure evidence-arithmetic-summary.csv.

## When to use

Use when auditing a review edition's reported corpus and screening counts (e.g. 38 screened / 11 admitted / 27 deferred) against companion repository artifacts to verify each published figure.

## When NOT to use

- The original interactive searches did not preserve every machine-issued query in a publication-ready log, so this skill cannot re-derive raw retrieval sessions — the companion distinguishes retained records from reconstructions
- Input is a live literature search rather than a frozen companion snapshot; the figures apply to the bounded update audit of Code Intelligence Digest editions (July 27 - August 5), not to new searching
- You need venue coverage beyond the artifact set: ACM Digital Library, IEEE Xplore, and Scopus searches were not performed and are a disclosed source limitation, not zero-result evidence

## Inputs

- companion/search-log.csv (retrieval queries and candidate counts, e.g. 148 OpenAlex candidates)
- companion/screening-decisions.csv (per-record screening outcomes: admitted, already present, deferred/excluded)
- companion/source-snapshot (evidence corpus composition: scholarly, practitioner, benchmark, author-system records)
- companion/publisher-coverage-status as JSON (venue coverage: SciX DOI matches, TSE known-set matches)
- companion/dblp-author-adjudication-2026-08.csv (DBLP publication adjudication cells)
- six retained-record files under companion/ in the cloned git repository

## Outputs

- evidence-arithmetic-summary.csv with one row per reported figure (figure name, value, source file)
- Reconciled screening outcome tally (38 screened; 11 admitted, 1 already present, 27 deferred-or-excluded)
- Venue-coverage figures (8-of-40 SciX DOI matches, 26-of-26 TSE known-set matches)
- DBLP tally (55 unique publications across 64 cells, 50 absent from prior sets)

## How to apply

Clone the companion repository (https://github.com/sjarmak/engineering-reliable-coding-agents.git) and locate the six retained-record files under companion/. Load the CSVs (search-log.csv, screening-decisions.csv, dblp-author-adjudication-2026-08.csv) and JSON files (publisher-coverage-status) with the Python 3 standard-library csv/ modules. From source-snapshot, tabulate the evidence corpus split (164 scholarly works, 100 practitioner records, 29 benchmark records, 17 author-system case records), and reconcile screening-decisions.csv into the outcome tally: 38 records screened, 11 admitted, 1 already present, 27 deferred-or-excluded. From search-log.csv and publisher-coverage-status compute retrieval coverage (148 OpenAlex candidates, 8-of-40 SciX DOI matches, 26-of-26 TSE known-set matches) and aggregate the DBLP adjudication file into 55 unique publications across 64 cells with 50 absent from prior sets. Write evidence-arithmetic-summary.csv with one row per reported figure (figure name, value, source file) and cross-check the catalog identity 206 = 193 gated practices + 13 research leads against companion/catalog.

## Related tools

- **git** (Clone the companion repository to obtain the retained-record files under companion/) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **Python 3** (Parse the CSV/JSON companion artifacts with the standard-library csv/ modules and write evidence-arithmetic-summary.csv)

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && python3 -c "import csv,; rows=list(csv.DictReader(open('companion/screening-decisions.csv'))); print(len(rows))"
```

## Evaluation signals

- Every tabulated value equals the reported figure (scholarly works = 164, practitioner records = 100, benchmark records = 29, author-system case records = 17)
- Screening reconciliation closes exactly: 38 = 11 admitted + 1 already present + 27 deferred-or-excluded
- Catalog identity check passes: 206 = 193 gated practices + 13 research leads against companion/catalog
- Coverage tallies match the published figures: 8-of-40 SciX DOI matches and 26-of-26 TSE known-set matches
- The output evidence-arithmetic-summary.csv parses with all expected fields present (figure name, value, source file)

## Limitations

- ACM Digital Library, IEEE Xplore, and Scopus searches were not performed; their absence is a disclosed source limitation, not an inference of zero relevant results
- The original interactive searches did not preserve every machine-issued query in a publication-ready log; the companion distinguishes retained records from reconstructions
- The bounded update audit covers only 38 distinct scholarly records from Code Intelligence Digest editions July 27 - August 5 plus one targeted check, so tallies do not generalize beyond that window

## Evidence

- [other] The bounded update audit screened 38 distinct scholarly records from Code Intelligence Digest editions (July 27 - August 5) plus one targeted check; of these, 11 new works were admitted, 1 record was already present, and 27 were deferred or excluded.: [evidence span withheld — anchor status: not_found]
- [methods] From source-snapshot tabulate the evidence corpus split: [evidence span withheld — anchor status: not_found]
- [methods] cross-check the catalog identity 206 = 193 gated practices + 13 research leads against companion/catalog: [evidence span withheld — anchor status: not_found]
