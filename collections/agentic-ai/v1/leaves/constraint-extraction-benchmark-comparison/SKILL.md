---
name: constraint-extraction-benchmark-comparison
description: Use when reproducing or evaluating LLM extraction of option conflicts/dependencies
  from man pages and you must score results against reported 633 constraints, 94.00%
  precision, 92.48% average precision, or the 20-program ground truth (80.56% recall).
when_to_use_negative:
- Your goal is downstream fuzzing evaluation (1748 predicted high-risk combinations,
  364 unique vulnerabilities, CVE counts) rather than validating the constraint-extraction
  module itself.
- No ground truth or paper-reported baseline (94.00% / 92.48% / 80.56%) exists for
  the corpus you are scoring, so precision and recall cannot be computed.
- Input documents are not command-line program documentation in Groff man-page form,
  so option conflicts and dependencies are not well defined.
edam_operation: http://edamontology.org/operation_2428
tools:
- name: ProphetFuzz
  role: Source of the extractRelationships and checkRelationships functions (llm_interface/constraint.py)
    whose output is benchmarked, plus the assets/dataset evaluation files
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM backend (gpt-4-1106-preview via OpenAI API) used for constraint extraction
    at temperature 0.7 (n=10) and the bidirectional self-check at temperature 0.2
- name: Python
  role: Runs the man-page parsing, extraction, per-program aggregation, and precision/recall
    comparison (~3k lines of Python code)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:3e5e95325d8c2d1b711c32f662f8960388d1ad047f099cd0b3a14e2784aac8ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# constraint-extraction-benchmark-comparison

## Summary

Benchmark a reproduced ProphetFuzz constraint-extraction run (LLM-extracted option conflicts/dependencies validated by bidirectional-reasoning self-check) against the paper's reported metrics: 633 constraints at 94.00% overall precision and 92.48% per-program average precision on 52 programs, plus 80.56% recall on the 180-constraint ground truth for the 20-program annotated subset.

## When to use

Use when reproducing or evaluating LLM extraction of option conflicts/dependencies from man pages and you must score results against reported 633 constraints, 94.00% precision, 92.48% average precision, or the 20-program ground truth (80.56% recall).

## When NOT to use

- Your goal is downstream fuzzing evaluation (1748 predicted high-risk combinations, 364 unique vulnerabilities, CVE counts) rather than validating the constraint-extraction module itself.
- No ground truth or paper-reported baseline (94.00% / 92.48% / 80.56%) exists for the corpus you are scoring, so precision and recall cannot be computed.
- Input documents are not command-line program documentation in Groff man-page form, so option conflicts and dependencies are not well defined.

## Inputs

- Groff man pages of the 52 evaluation programs (from 40 packages), input to manpage_parser
- LLM-extracted conflict/dependency constraints produced by extractRelationships + checkRelationships (GPT-4 Turbo, gpt-4-1106-preview, OpenAI API)
- assets/dataset/groundtruth_for_20_programs. — 180 ground-truth constraints for the 20-program annotated subset
- assets/dataset/precision. — paper precision values for the constraint-extraction module

## Outputs

- Per-program constraint table (constraints_per_program.csv) with only self-check-surviving conflicts and dependencies
- Benchmark report: extracted-constraint counts, overall precision, and per-program average precision on the 52-program set vs reported 633 / 94.00% / 92.48%
- Recall figure for the 20-program annotated subset vs the reported 80.56%, with a pass/fail validation verdict

## How to apply

Parse the Groff man pages of the 52 programs with manpage_parser (program name via .TH, sections via .SH, option descriptions via .TP/.PP/.RS/.sp), then run extractRelationships in llm_interface/constraint.py with GPT-4 Turbo (gpt-4-1106-preview) at temperature 0.7 and n=10 inferences per call. Apply the checkRelationships bidirectional-reasoning self-check at temperature 0.2 with a self-check voting threshold of 5 so only constraints surviving the filter enter the benchmark. Aggregate surviving conflicts and dependencies into a per-program table (constraints_per_program.csv), count constraints, and compare the 52-program totals and judged precision against the paper's 633 constraints, 94.00% overall precision, and 92.48% per-program average precision (diffing per-program values against assets/dataset/precision.). For the 20-program annotated subset, score recall against assets/dataset/groundtruth_for_20_programs. (180 ground-truth constraints) and compare with the reported 80.56%. Judge the reproduction correct only if overall precision is near 94.00%, per-program average precision near 92.48%, and 20-program recall is comparable to 80.56% — Carpet Fuzz (the weaker baseline) reported lower precision, so large deviations signal a pipeline or LLM-version mismatch.

## Related tools

- **ProphetFuzz** (Source of the extractRelationships and checkRelationships functions (llm_interface/constraint.py) whose output is benchmarked, plus the assets/dataset evaluation files) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM backend (gpt-4-1106-preview via OpenAI API) used for constraint extraction at temperature 0.7 (n=10) and the bidirectional self-check at temperature 0.2)
- **Python** (Runs the man-page parsing, extraction, per-program aggregation, and precision/recall comparison (~3k lines of Python code))

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Extracted-constraint count across the 52 programs is close to the reported 633 constraints.
- Judged overall precision is near 94.00% and per-program average precision near 92.48%.
- Recall on the 20-program annotated subset, computed against groundtruth_for_20_programs. (180 constraints), is comparable to the reported 80.56%.
- Only constraints surviving the checkRelationships self-check (temperature 0.2, voting threshold 5) appear in constraints_per_program.csv.
- Per-program precision values are consistent with assets/dataset/precision. when diffed.

## Limitations

- LLM hallucination threatens precision of extracted constraints; the bidirectional self-check filters incorrect constraints but the reported overall precision is 94.00%, not 100%.
- Recall can only be benchmarked on the 20-program annotated subset (180 ground-truth constraints); the other programs in the 52-program set lack ground truth.
- Sampling parameters (temperature 0.7, n=10 per call on gpt-4-1106-preview) introduce run-to-run nondeterminism and per-program OpenAI API cost; the benchmarked configuration is tied to GPT-4 Turbo.

## Evidence

- [results] With the self-check method, ProphetFuzz extracted 633 constraints from the program documentation at an overall precision of 94.00% and an average precision of 92.48%, outperforming Carpet Fuzz.: [evidence span withheld — anchor status: not_found]
- [methods] Run constraint extraction (extractRelationships in llm_interface/constraint.py) with GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API, setting temperature to 0.7: [evidence span withheld — anchor status: not_found]
- [methods] Apply the bidirectional-reasoning self-check (checkRelationships) with temperature 0.2 and a self-check voting threshold of 5 to filter incorrect constraints.: [evidence span withheld — anchor status: not_found]
- [methods] for the 20-program annotated subset, against groundtruth_for_20_programs. (180 ground-truth constraints, reported recall 80.56%): [evidence span withheld — anchor status: not_found]
- [methods] the reproduced constraint table is judged correct if the overall precision is near the reported 94.00% and per-program average precision near 92.48%: [evidence span withheld — anchor status: not_found]
- [readme] assets/dataset: Dataset for eveluating constraint extraction module.: "assets/dataset: Dataset for eveluating constraint extraction module."
