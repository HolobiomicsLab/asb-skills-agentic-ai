---
name: claim-to-source-ledger-joining
description: Use when a machine-readable catalog of gated practices with a stable
  ID scheme must be joined to a separate claim-to-source evidence ledger to test whether
  evidence strength tracks an ordering variable.
when_to_use_negative:
- The practices of interest have no consequence/urgency ranking, so no ordering variable
  exists to correlate against the evidence indicator.
- The analysis needs the full 206-record catalog (193 gated practices plus 13 research
  leads) rather than the 52-practice consequence-ranking subset.
- Evidence-group assignments must be re-adjudicated rather than taken as given — this
  skill consumes the ledger's strong/weak grouping, it does not re-grade evidence.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: git
  role: Clone https://github.com/sjarmak/engineering-reliable-coding-agents.git to
    obtain companion/catalog and companion/evidence-ledger.csv
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: Python 3
  role: Parse the catalog, join it to the evidence ledger, encode the binary strong-evidence
    indicator, and compute the Spearman correlation with average ranks for ties
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: sha256sum
  role: Verify companion-directory integrity before joining catalog and ledger records
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:b2730cc6b8c3af997874e344b4d4f5c091dded1a50dc1f1a1477edbbafe6917a
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
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Claim-to-source ledger joining

## Summary

Join a versioned practice catalog (keyed by stable ERCA-NNN identifiers) to a claim-to-source evidence ledger in CSV, encode a per-practice binary strong-evidence indicator, and correlate it against an operational ranking with Spearman rank correlation (here yielding -0.004 over n=52).

## When to use

Use when a machine-readable catalog of gated practices with a stable ID scheme must be joined to a separate claim-to-source evidence ledger to test whether evidence strength tracks an ordering variable.

## When NOT to use

- The practices of interest have no consequence/urgency ranking, so no ordering variable exists to correlate against the evidence indicator.
- The analysis needs the full 206-record catalog (193 gated practices plus 13 research leads) rather than the 52-practice consequence-ranking subset.
- Evidence-group assignments must be re-adjudicated rather than taken as given — this skill consumes the ledger's strong/weak grouping, it does not re-grade evidence.

## Inputs

- companion/catalog (machine-readable catalog of gated practices with stable ERCA-NNN identifiers and consequence/urgency ranks)
- companion/evidence-ledger.csv (claim-to-source links including evidence-group assignments)
- SHA256SUMS checksum manifest for the companion directory

## Outputs

- Per-practice binary has-at-least-one-strong-evidence indicator keyed by ERCA-NNN
- spearman_correlation file containing the rank correlation coefficient and sample size n
- Validation statement that n equals the ranking-subset size (52) and the coefficient lies within [-1, 1]

## How to apply

Clone the sjarmak/engineering-reliable-coding-agents repository with git to obtain companion/catalog and companion/evidence-ledger.csv, optionally verifying companion file integrity with sha256sum -c SHA256SUMS. Parse companion/catalog with Python 3 to extract the gated practices that carry consequence/urgency rankings, keyed by their stable ERCA-NNN identifiers. Join the catalog records to companion/evidence-ledger.csv claim-to-source links and encode a binary indicator per practice: 1 if the practice has at least one evidence item in the strong group (direct support via controlled comparison, validated benchmark result, or comparably specific measurement), else 0. Compute the Spearman rank correlation between urgency rank and the binary indicator in Python 3, applying average ranks for ties introduced by the binary variable, and write the coefficient and sample size n to spearman_correlation.

## Related tools

- **git** (Clone https://github.com/sjarmak/engineering-reliable-coding-agents.git to obtain companion/catalog and companion/evidence-ledger.csv) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **Python 3** (Parse the catalog, join it to the evidence ledger, encode the binary strong-evidence indicator, and compute the Spearman correlation with average ranks for ties) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **sha256sum** (Verify companion-directory integrity before joining catalog and ledger records) — https://github.com/sjarmak/engineering-reliable-coding-agents

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && (cd engineering-reliable-coding-agents/companion && sha256sum -c SHA256SUMS) && python3 scripts/check_release_counts.py
```

## Evaluation signals

- The join key space is stable ERCA-NNN identifiers and every matched record resolves to a single catalog practice.
- The computed n equals the 52-practice consequence-ranking subset size.
- The Spearman coefficient lies within [-1, 1]; the reported value for this dataset is -0.004.
- Ties introduced by the binary indicator are handled with average ranks in the rank calculation.
- companion file integrity passes sha256sum -c SHA256SUMS before the join is performed.

## Limitations

- The result is restricted to the 52-practice consequence-ranking subset, not all 193 gated practices or the 206 total reliability records.
- Evidence strength varies by topic in the source monograph, so the strong/weak grouping reflects the ledger's admission gates and is not a uniform measurement quality guarantee.
- Empirical results of the underlying catalog remain dependent on workload and system configuration, so the near-zero correlation should not be generalized to other catalogs or ranking schemes.

## Evidence

- [abstract] The monograph contributes a versioned catalog of 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads: "The monograph contributes a versioned catalog of 206 reliability records: 193 gated practices, including 56 developed in depth, plus 13 research leads"
- [intro] practice admission gate: "A record qualified at the final gate through at least one scholarly item, a non-author synthesis with a resolvable scholarly identity, or at least two practitioner items"
- [methods] git clone: "git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git"
- [methods] sha256sum verify: "(cd companion && sha256sum -c SHA256SUMS)"
- [readme] machine-readable evidence and practice catalog: "This repository contains the technical review and engineering monograph, runnable reliability and evaluation protocols, a machine-readable evidence and practice catalog, and reusable agent skills"
