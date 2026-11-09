---
name: cohens-kappa-agreement-analysis
description: Use when a release supplies a blinded rating packet (labels hidden) plus
  a reader response template and you must quantify inter-rater agreement (observed
  agreement, Cohen's kappa, Fleiss's kappa) between your grades and hidden reference
  labels.
when_to_use_negative:
- The task already provides a published inter-rater agreement value or calibration
  report — this skill exists because the edition reports none and does not claim reproducibility
  of its evidence-group assignments
- The input is not a categorical grading task with mutually assignable grade categories;
  kappa statistics are defined for categorical ratings, not continuous scores
- The analysis requires completed external grader data that does not exist — no completed
  external grader response or calibration report is part of this edition, so agreement
  can only be measured by running the blinded pass yourself
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: git
  role: Clone sjarmak/engineering-reliable-coding-agents to obtain the blinded packet,
    review form, hidden labels, and the source of the worked example
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: Node.js
  role: Runtime for analyze-grades.mjs, which parses reader grades and computes observed
    agreement, pairwise Cohen's kappa, Fleiss's kappa, and disagreement patterns
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:3feab04ba9ab96bc8678e3bde943d710499f6267baa4995bc9af02e5cd796d8e
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents
  tools:
  - Node.js
  - git
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cohens-kappa-agreement-analysis

## Summary

A Node.js agreement-analysis skill that computes raw observed agreement, pairwise Cohen's kappa for every reader pair, and Fleiss's kappa across all readers on a blinded grading packet, validating the implementation first against the chapter's 100-item worked example. It lets external readers measure inter-rater agreement themselves because this edition reports no agreement value of its own.

## When to use

Use when a release supplies a blinded rating packet (labels hidden) plus a reader response template and you must quantify inter-rater agreement (observed agreement, Cohen's kappa, Fleiss's kappa) between your grades and hidden reference labels.

## When NOT to use

- The task already provides a published inter-rater agreement value or calibration report — this skill exists because the edition reports none and does not claim reproducibility of its evidence-group assignments
- The input is not a categorical grading task with mutually assignable grade categories; kappa statistics are defined for categorical ratings, not continuous scores
- The analysis requires completed external grader data that does not exist — no completed external grader response or calibration report is part of this edition, so agreement can only be measured by running the blinded pass yourself

## Inputs

- Blinded 20-practice grading packet with the author's labels hidden (from the sjarmak/engineering-reliable-coding-agents release artifact)
- Reviewer response template (review-form.html) filled with reader grades
- Author's hidden reference labels for the 20 sampled practices
- Chapter's 100-item worked example (reference agreement data used as a regression fixture)

## Outputs

- agreement_report. containing observed agreement, all pairwise Cohen's kappa values, and three-reader Fleiss's kappa
- Structured disagreement-pattern breakdown tabulated by grade-category mismatch

## How to apply

Clone the sjarmak/engineering-reliable-coding-agents repository with git and locate the grading packet files inside the checkout: the blinded deterministic random sample of 20 practices with the author's labels hidden, review-form.html, and the author's hidden reference labels. Implement analyze-grades.mjs in Node.js to parse the reader grade records, compute raw observed agreement, pairwise Cohen's kappa for each reader pair, Fleiss's kappa across all three readers, and tabulate disagreement patterns by grade-category mismatch. Before trusting the implementation, run it in regression mode against the chapter's 100-item worked example and assert it reproduces 90% observed agreement and kappa ≈ 0.44; only then execute it on the blinded 20-practice packet, comparing each reader's grades against the author's hidden labels and each other. Emit a complete agreement_report. covering all 20 packet practices with observed agreement, all pairwise kappa values, the three-reader Fleiss's kappa, and a structured disagreement-pattern breakdown.

## Related tools

- **git** (Clone sjarmak/engineering-reliable-coding-agents to obtain the blinded packet, review form, hidden labels, and the source of the worked example) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **Node.js** (Runtime for analyze-grades.mjs, which parses reader grades and computes observed agreement, pairwise Cohen's kappa, Fleiss's kappa, and disagreement patterns) — https://github.com/sjarmak/engineering-reliable-coding-agents

## Examples

```
node analyze-grades.mjs
```

## Evaluation signals

- Regression-mode run against the chapter's 100-item worked example reproduces observed agreement = 0.90 and kappa ≈ 0.44 before the packet is analyzed
- agreement_report. is complete and covers all 20 packet practices
- Every reader pair has a pairwise Cohen's kappa value and the report includes a single three-reader Fleiss's kappa
- Observed agreement, kappa, and disagreement-pattern counts are mutually consistent (e.g. kappa < observed agreement, as in the worked example)
- Disagreement-pattern breakdown accounts for every grade-category mismatch found in the packet

## Limitations

- This edition has not commissioned external graders, reports no inter-rater agreement value, and does not claim independent calibration or reproducibility — the skill measures agreement on a blinded sample rather than reproducing a published number
- The packet is a deterministic random sample of only 20 practices, so kappa estimates from a single small blinded pass carry sampling uncertainty
- The implementation must be validated against the worked example first; an unvalidated kappa implementation can silently miscompute chance-adjusted agreement

## Evidence

- [intro] This edition has not commissioned external graders, reports no inter-rater agreement value, and does not claim independent calibration or reproducibility: "This edition has not commissioned external graders, reports no inter-rater agreement value, and does not claim independent calibration or reproducibility"
- [methods] git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git: "git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git"
- [methods] node scripts/new-protocol-run.mjs minimum-reliability-pass ./runs/my-agent: "node scripts/new-protocol-run.mjs minimum-reliability-pass ./runs/my-agent"
