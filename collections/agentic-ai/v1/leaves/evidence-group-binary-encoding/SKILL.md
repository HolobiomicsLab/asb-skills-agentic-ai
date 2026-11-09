---
name: evidence-group-binary-encoding
description: Use when a versioned practice catalog with urgency rankings and a claim-to-source
  evidence ledger need to be joined and reduced to a per-practice binary evidence-strength
  indicator for rank correlation.
when_to_use_negative:
- The practices under study carry no consequence/urgency ranking, so there is no rank
  variable to correlate against
- The evidence ledger lacks evidence-group assignments, so a `strong` group cannot
  be distinguished from weaker items
- The analysis needs graded evidence-strength scores rather than a binary strong/non-strong
  encoding
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: git
  role: Clone the sjarmak/engineering-reliable-coding-agents repository to obtain
    companion/catalog. and companion/evidence-ledger.csv
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: Python 3
  role: Parse the catalog, join evidence-ledger.csv links, encode the binary indicator,
    and compute the Spearman rank correlation with average ranks for ties
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:0a9c7736bc631f16d8247925d955d24571a41af59f29190a320b567b8b29866a
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents
  tools:
  - Python
  - Python 3
  - git
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# evidence-group-binary-encoding

## Summary

Encodes each gated practice from the companion catalog as a binary indicator (1 if it carries at least one 'strong'-group evidence item, else 0) and correlates that indicator with the practice's consequence/urgency rank, exposing whether evidence strength tracks operational urgency across the catalog.

## When to use

Use when a versioned practice catalog with urgency rankings and a claim-to-source evidence ledger need to be joined and reduced to a per-practice binary evidence-strength indicator for rank correlation.

## When NOT to use

- The practices under study carry no consequence/urgency ranking, so there is no rank variable to correlate against
- The evidence ledger lacks evidence-group assignments, so a `strong` group cannot be distinguished from weaker items
- The analysis needs graded evidence-strength scores rather than a binary strong/non-strong encoding

## Inputs

- companion/catalog. from the sjarmak/engineering-reliable-coding-agents repository (gated practices with consequence/urgency rankings and ERCA-NNN identifiers)
- companion/evidence-ledger.csv (claim-to-source links with evidence-group assignments)

## Outputs

- Per-practice binary has-at-least-one-strong-evidence indicator keyed by ERCA-NNN identifier
- spearman_correlation. file containing the Spearman coefficient and sample size n

## How to apply

Clone sjarmak/engineering-reliable-coding-agents with git to obtain companion/catalog. and companion/evidence-ledger.csv. Parse the catalog with Python 3 to extract the gated practices that carry consequence/urgency rankings, keyed by stable ERCA-NNN identifiers. Join catalog records to the evidence-ledger claim-to-source links and set the binary to 1 when a practice has at least one evidence item in the `strong` group (direct support via controlled comparison, validated benchmark result, or comparably specific measurement), else 0. Compute Spearman's rank correlation between urgency rank and the binary indicator with Python 3, applying average ranks for ties introduced by the binary variable, then write the coefficient and n to spearman_correlation..

## Related tools

- **git** (Clone the sjarmak/engineering-reliable-coding-agents repository to obtain companion/catalog. and companion/evidence-ledger.csv) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **Python 3** (Parse the catalog, join evidence-ledger.csv links, encode the binary indicator, and compute the Spearman rank correlation with average ranks for ties)

## Examples

```
python3 -c "from scipy.stats import spearmanr; rho, p = spearmanr(urgency_rank, strong_indicator)"  # urgency_rank, strong_indicator parsed from companion/catalog. joined to companion/evidence-ledger.csv
```

## Evaluation signals

- n equals the 52-practice consequence-ranking subset size
- The Spearman coefficient lies within [-1, 1]; the published reproduction yields -0.004
- Every practice in the ranking subset is keyed by a stable ERCA-NNN identifier with no unmatched or dropped records after the join to evidence-ledger.csv
- Tied values of the binary indicator receive average ranks before rank correlation is computed
- The indicator is 1 exactly for practices with at least one `strong`-group item (direct support via controlled comparison, validated benchmark result, or comparably specific measurement)

## Limitations

- The resulting correlation is on a 52-practice subset of the 193 gated practices, so it does not characterize the full catalog
- Evidence strength varies by topic and results depend on workload and system configuration; the binary collapse discards within-group strength differences
- The original interactive searches did not preserve every machine-issued query in a publication-ready log; the companion distinguishes retained records from reconstructions

## Evidence

- [abstract] Catalog scale context: [evidence span withheld — anchor status: not_found]
