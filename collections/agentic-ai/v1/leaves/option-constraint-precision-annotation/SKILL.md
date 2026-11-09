---
name: option-constraint-precision-annotation
description: 'Use when reproducing or auditing ProphetFuzz''s constraint-extraction
  module: you need to measure precision/recall of option constraints extracted by
  GPT-4 Turbo from man pages against ground-truth annotations.'
when_to_use_negative:
- The goal is high-risk combination prediction, command assembly, or fuzzing rather
  than evaluating constraint extraction precision (use llm_interface/predict.py and
  assembly.py pipelines instead).
- Program documentation is not in Groff man-page format, since the parser (.TH/.SH/.TP/.PP/.RS/.sp
  keyword matching) assumes Groff structure.
- No ground truth or paper-reported precision values are available for comparison
  and no manual annotation can be performed.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: ProphetFuzz
  role: Source framework providing manpage_parser, extractRelationships, and checkRelationships
    used for constraint extraction and self-check filtering
  repo: github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM backend (gpt-4-1106-preview) used for both constraint extraction (temp
    0.7, n=10) and bidirectional self-check (temp 0.2, voting threshold 5)
- name: Python
  role: Implementation language of the constraint extraction and self-check pipeline
    (~3k lines) and of the aggregation/benchmark scripts
  repo: github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:163a246a1ac50191ea87eb09adfd6df9350d2587feb7ca9c90fc36d39ad9ff1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# option-constraint-precision-annotation

## Summary

A method for evaluating the precision of LLM-extracted option constraints (conflicts and dependencies) from Groff man pages, using ProphetFuzz's bidirectional-reasoning self-check to filter hallucinated constraints and benchmarking against paper-reported metrics.

## When to use

Use when reproducing or auditing ProphetFuzz's constraint-extraction module: you need to measure precision/recall of option constraints extracted by GPT-4 Turbo from man pages against ground-truth annotations.

## When NOT to use

- The goal is high-risk combination prediction, command assembly, or fuzzing rather than evaluating constraint extraction precision (use llm_interface/predict.py and assembly.py pipelines instead).
- Program documentation is not in Groff man-page format, since the parser (.TH/.SH/.TP/.PP/.RS/.sp keyword matching) assumes Groff structure.
- No ground truth or paper-reported precision values are available for comparison and no manual annotation can be performed.

## Inputs

- Groff man pages for the 52 programs from 40 packages (evaluation dataset from three related studies)
- OpenAI API key configured in llm_interface/config/.env
- Ground-truth annotations: assets/dataset/groundtruth_for_20_programs. (180 constraints for 20 programs)
- Paper-reported reference values: 633 constraints, 94.00% overall precision, 92.48% average precision, 80.56% recall on 20 programs

## Outputs

- Per-program constraint table (conflicts and dependencies) e.g. constraints_per_program.csv
- Overall and per-program precision estimates for the extracted constraint set
- Recall estimate for the 20-program annotated subset
- Filtered constraint set that survived the self-check voting threshold

## How to apply

Parse the Groff man pages of the 52 programs with manpage_parser (program name via .TH, sections via .SH, option descriptions via .TP/.PP/.RS/.sp) to obtain structured option descriptions. Run constraint extraction via extractRelationships in llm_interface/constraint.py with GPT-4 Turbo (gpt-4-1106-preview) at temperature 0.7 and n=10 inferences per call, then apply the bidirectional-reasoning self-check (checkRelationships) at temperature 0.2 with a voting threshold of 5 to filter incorrect constraints. Aggregate surviving conflicts and dependencies into a per-program constraint table (constraints_per_program.csv). Benchmark against the paper's reported 633 constraints at 94.00% overall precision and 92.48% average per-program precision, and, for the 20-program annotated subset, against assets/dataset/groundtruth_for_20_programs. (180 ground-truth constraints, reported recall 80.56%). The rationale is that LLMs tend to flag option combinations that violate constraints as high-risk, so constraint precision directly gates downstream high-risk combination prediction.

## Related tools

- **ProphetFuzz** (Source framework providing manpage_parser, extractRelationships, and checkRelationships used for constraint extraction and self-check filtering) — github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM backend (gpt-4-1106-preview) used for both constraint extraction (temp 0.7, n=10) and bidirectional self-check (temp 0.2, voting threshold 5))
- **Python** (Implementation language of the constraint extraction and self-check pipeline (~3k lines) and of the aggregation/benchmark scripts) — github.com/NASP-THU/ProphetFuzz

## Examples

```
python llm_interface/constraint.py && bash run_all_in_one.sh bison  # set OPENAI_API_KEY in llm_interface/config/.env first
```

## Evaluation signals

- Overall precision of the extracted constraint set is near the reported 94.00%.
- Average per-program precision is near the reported 92.48%.
- Total surviving constraints across the 52 programs is near the reported count of 633.
- Recall on the 20-program annotated subset (groundtruth_for_20_programs., 180 constraints) is comparable to the reported 80.56%.
- Every extracted constraint passed the self-check: bidirectional verification and counterexample questions with voting threshold 5 out of n=10 inferences.

## Limitations

- LLM hallucination threatens constraint-extraction precision, which is why the self-check is essential; residual errors persist after filtering.
- Precision/recall verification against ground truth is only possible for the 20-program annotated subset, not all 52 programs.
- Results depend on a proprietary API model (gpt-4-1106-preview) and specific sampling settings (temperature 0.7 with n=10 for extraction, temperature 0.2 for self-check), so reproducibility may drift with model updates or cost budgets.

## Evidence

- [intro] Challenge C1 motivating constraint precision: [evidence span withheld — anchor status: not_found]
