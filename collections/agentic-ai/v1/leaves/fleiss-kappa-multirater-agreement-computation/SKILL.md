---
name: fleiss-kappa-multirater-agreement-computation
description: Use when multiple readers grade the same categorical items (e.g., a blinded
  practice packet) and you need observed agreement, pairwise Cohen's kappa, and Fleiss's
  kappa to quantify inter-rater agreement.
when_to_use_negative:
- Fewer than two raters or a single reader's grades — pairwise Cohen's kappa and three-reader
  Fleiss's kappa are undefined without multiple independent readers
- Grades are continuous scores or ranked magnitudes rather than categorical grade
  categories — the analysis tabulates disagreement by grade-category mismatch, which
  presumes nominal categories
- The goal is reproducibility of agent runs or test outcomes rather than agreement
  between human/reader graders over a labeled item set
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Node.js
  role: Runtime for implementing and executing analyze-grades.mjs, which computes
    observed agreement, pairwise Cohen's kappa, Fleiss's kappa, and disagreement patterns
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
- name: git
  role: Used to clone the sjarmak/engineering-reliable-coding-agents checkout and
    locate the blinded 20-practice packet, review-form.html, and hidden reference
    labels
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:e5b4f4dfd0b1b2192bc7f0a4f51ef3c41423c31c4f00580984d716ab196a4056
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents.git
  tools:
  - Node.js
  - git
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fleiss-kappa-multirater-agreement-computation

## Summary

A Node.js-based agreement analysis skill that parses multi-reader grade records over a blinded packet, computes raw observed agreement, pairwise Cohen's kappa for each reader pair, and Fleiss's kappa across all readers, then emits a structured agreement report with disagreement patterns by grade-category mismatch.

## When to use

Use when multiple readers grade the same categorical items (e.g., a blinded practice packet) and you need observed agreement, pairwise Cohen's kappa, and Fleiss's kappa to quantify inter-rater agreement.

## When NOT to use

- Fewer than two raters or a single reader's grades — pairwise Cohen's kappa and three-reader Fleiss's kappa are undefined without multiple independent readers
- Grades are continuous scores or ranked magnitudes rather than categorical grade categories — the analysis tabulates disagreement by grade-category mismatch, which presumes nominal categories
- The goal is reproducibility of agent runs or test outcomes rather than agreement between human/reader graders over a labeled item set

## Inputs

- Blinded random sample of 20 practices with the author's labels hidden
- Reader grade records (completed review-form.html reviewer response template)
- Author's hidden reference labels for the packet practices
- Chapter's 100-item worked example with known reference agreement values (observed agreement = 0.90, kappa ~0.44)

## Outputs

- agreement_report. with observed agreement
- Pairwise Cohen's kappa values for each reader pair
- Three-reader Fleiss's kappa
- Structured disagreement-pattern breakdown by grade-category mismatch

## How to apply

Clone the sjarmak/engineering-reliable-coding-agents repository and locate the grading packet files: the blinded 20-practice packet (author's labels hidden), review-form.html (the reviewer response template), and the author's hidden reference labels. Implement analyze-grades.mjs in Node.js to parse the reader grade records, compute raw observed agreement, pairwise Cohen's kappa for each reader pair, and Fleiss's kappa across all three readers, and tabulate disagreement patterns by grade-category mismatch. Before trusting the implementation, validate it in regression mode against the chapter's 100-item worked example and assert it reproduces the reference values of 0.90 observed agreement and kappa ~0.44. Only then run the validated script on the blinded 20-practice packet, comparing each reader's grades against the author's hidden labels and each other. Emit agreement_report. containing observed agreement, all pairwise Cohen's kappa values, the three-reader Fleiss's kappa, and a structured disagreement-pattern breakdown.

## Related tools

- **Node.js** (Runtime for implementing and executing analyze-grades.mjs, which computes observed agreement, pairwise Cohen's kappa, Fleiss's kappa, and disagreement patterns) — https://github.com/sjarmak/engineering-reliable-coding-agents.git
- **git** (Used to clone the sjarmak/engineering-reliable-coding-agents checkout and locate the blinded 20-practice packet, review-form.html, and hidden reference labels) — https://github.com/sjarmak/engineering-reliable-coding-agents.git

## Examples

```
node analyze-grades.mjs --regression && node analyze-grades.mjs --input blinded-20-practice-packet --labels hidden-reference-labels
```

## Evaluation signals

- Regression mode must reproduce the worked-example reference values: observed agreement = 0.90 and kappa ~0.44 on the 100-item worked example, before the packet run is trusted
- agreement_report. must cover all 20 packet practices with no missing reader grades
- The report contains all pairwise Cohen's kappa values (one per reader pair) plus the three-reader Fleiss's kappa
- The disagreement-pattern breakdown is structured by grade-category mismatch, not just a single agreement number
- Reader grades are compared both against the author's hidden reference labels and against each other

## Limitations

- This edition of the monograph has not commissioned external graders, reports no inter-rater agreement value, and does not claim independent calibration or reproducibility — the tooling exists for external readers to measure agreement themselves
- The worked example only reaches kappa ~0.44 at 0.90 observed agreement, illustrating that raw agreement can substantially overstate chance-corrected agreement; interpret kappa accordingly
- The blinded packet is a deterministic random sample of 20 practices, so agreement statistics from it carry small-sample uncertainty and do not cover the full 206-record catalog

## Evidence

- [other] The release artifact includes a deterministic random sample of 20 practices with the author's labels hidden, a reviewer response template, and a script that can report pairwise Cohen's kappa: [evidence span withheld — anchor status: not_found]
