---
name: spearman-rank-correlation-computation
description: Use when two per-record attributes from a structured catalog must be
  compared by rank — e.g. an urgency/consequence rank versus a binary evidence-strength
  indicator — to quantify their association.
when_to_use_negative:
- The records of interest are not gated practices carrying consequence/urgency rankings
  (e.g. the 13 research leads rather than the ranking subset)
- The question requires causal inference or a coefficient-free comparison rather than
  a rank-association statistic
- The evidence ledger lacks resolvable claim-to-source links, so the strong-evidence
  indicator cannot be constructed
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: git
  role: Clone sjarmak/engineering-reliable-coding-agents to obtain companion/catalog.
    and companion/evidence-ledger.csv
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: Python 3
  role: Parse the catalog, join evidence-ledger links, encode the binary strong-evidence
    indicator, and compute the Spearman correlation with average ranks for ties
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:47317a77be22eccbf8d5e27c1e30d3ef9f5deec0e708aafbbcd43edfd95be4d8
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spearman-rank-correlation-computation

## Summary

Computes a Spearman rank correlation between a ranked (ordinal) variable and a derived per-record indicator across a gated practice subset, quantifying whether two catalog attributes co-vary. In this catalog it tests whether operational urgency rank correlates with carrying at least one strong evidence item.

## When to use

Use when two per-record attributes from a structured catalog must be compared by rank — e.g. an urgency/consequence rank versus a binary evidence-strength indicator — to quantify their association.

## When NOT to use

- The records of interest are not gated practices carrying consequence/urgency rankings (e.g. the 13 research leads rather than the ranking subset)
- The question requires causal inference or a coefficient-free comparison rather than a rank-association statistic
- The evidence ledger lacks resolvable claim-to-source links, so the strong-evidence indicator cannot be constructed

## Inputs

- companion/catalog. (machine-readable practice catalog with consequence/urgency rankings and ERCA-NNN identifiers)
- companion/evidence-ledger.csv (claim-to-source links with evidence groups, including the `strong` group)

## Outputs

- spearman_correlation. containing the Spearman correlation coefficient and sample size n
- Binary has-at-least-one-strong-evidence indicator per ERCA-NNN practice

## How to apply

Clone sjarmak/engineering-reliable-coding-agents with git to obtain companion/catalog. and companion/evidence-ledger.csv, then parse the catalog with Python 3 to extract the gated practices carrying consequence/urgency rankings keyed by their stable ERCA-NNN identifiers. Join catalog records to the evidence-ledger claim-to-source links and encode a binary indicator per practice: 1 if the practice has at least one evidence item in the `strong` group (direct support via controlled comparison, validated benchmark result, or comparably specific measurement), else 0. Compute the Spearman rank correlation between urgency rank and the binary indicator in Python 3, applying average ranks for ties introduced by the binary variable. Write the correlation coefficient and sample size n to spearman_correlation., then validate that n equals the ranking-subset size (52 practices here) and that the coefficient lies within [-1, 1].

## Related tools

- **git** (Clone sjarmak/engineering-reliable-coding-agents to obtain companion/catalog. and companion/evidence-ledger.csv) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **Python 3** (Parse the catalog, join evidence-ledger links, encode the binary strong-evidence indicator, and compute the Spearman correlation with average ranks for ties) — https://github.com/sjarmak/engineering-reliable-coding-agents

## Examples

```
python3 -c "from scipy.stats import spearmanr; rho,_ = spearmanr(urgency_rank, has_strong_evidence); print(rho, len(urgency_rank))"
```

## Evaluation signals

- n equals the 52-practice consequence-ranking subset size
- Correlation coefficient lies within [-1, 1]; the reported value is -0.004, indicating near-zero association
- Every practice in the ranking subset maps to a stable ERCA-NNN identifier with no missing join to the evidence ledger
- Ties in the binary indicator are handled with average ranks, so duplicate ranks receive the mean of their rank positions
- Indicator assignment matches the ledger: 1 only when at least one linked evidence item is in the `strong` group

## Limitations

- The result reflects only the 52-practice ranking subset of 193 gated practices; it does not generalize to all catalog records
- A near-zero Spearman coefficient (-0.004) indicates absence of monotone rank association, not absence of any relationship
- The binary indicator collapses heterogeneous strong evidence (controlled comparisons, validated benchmark results, comparably specific measurements) into one value

## Evidence
