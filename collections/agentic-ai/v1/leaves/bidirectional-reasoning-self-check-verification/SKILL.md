---
name: bidirectional-reasoning-self-check-verification
description: Use when LLM-extracted option constraints (conflicts/dependencies) from
  command-line documentation must be verified against hallucination before guiding
  high-risk combination prediction or fuzz-target selection.
when_to_use_negative:
- The task is to discover new constraints rather than verify existing LLM candidates
  — use extractRelationships (constraint extraction) instead; this skill only accepts/rejects
  candidates.
- Documentation is not Groff/man-page format (no .TP/.PP/.RS/.sp option descriptions),
  so the extraction pipeline feeding this check cannot produce reliable candidates.
- Constraint errors carry no downstream cost (e.g., no option-combination prediction
  follows), in which case the repeated self-check LLM calls add API cost without benefit.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ProphetFuzz
  role: Hosts checkRelationships (the bidirectional-reasoning self-check) and extractRelationships
    in llm_interface/constraint.py; checkRelationships filters candidates by the voting
    threshold
  repo: github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: 'The LLM interrogated by the self-check: extraction at temperature 0.7 with
    n=10 calls, self-check verification/counterexample questions at temperature 0.2
    (gpt-4-1106-preview via OpenAI API)'
- name: manpage_parser
  role: Parses Groff man pages (.TH/.SH/.TP/.PP/.RS/.sp) into the structured option
    descriptions that both candidate extraction and self-check questions are grounded
    in
  repo: github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:dbed645b2c930b86ff3c9c5290269cc2484d84fe189a7c12017f92cd7b8f8ac5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - manpage_parser
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bidirectional-reasoning-self-check-verification

## Summary

A bidirectional-reasoning self-check that re-interrogates GPT-4 Turbo (gpt-4-1106-preview) with verification plus counterexample questions and retains an option constraint only when the self-check voting threshold of 5 is met, filtering hallucinated conflicts/dependencies extracted from Groff man pages so precision reaches 94.00% overall.

## When to use

Use when LLM-extracted option constraints (conflicts/dependencies) from command-line documentation must be verified against hallucination before guiding high-risk combination prediction or fuzz-target selection.

## When NOT to use

- The task is to discover new constraints rather than verify existing LLM candidates — use extractRelationships (constraint extraction) instead; this skill only accepts/rejects candidates.
- Documentation is not Groff/man-page format (no .TP/.PP/.RS/.sp option descriptions), so the extraction pipeline feeding this check cannot produce reliable candidates.
- Constraint errors carry no downstream cost (e.g., no option-combination prediction follows), in which case the repeated self-check LLM calls add API cost without benefit.

## Inputs

- Structured option descriptions parsed from Groff man pages (manpage_parser output: program name via .TH, sections via .SH, option descriptions via .TP/.PP/.RS/.sp)
- Candidate constraint list (conflicts and dependencies) produced by extractRelationships in llm_interface/constraint.py (GPT-4 Turbo, temperature 0.7, n=10 per call)
- OpenAI API key configured in llm_interface/config/.env
- Evaluation data: assets/dataset/groundtruth_for_20_programs. and precision. (20-program annotated subset, 180 ground-truth constraints)

## Outputs

- Filtered per-program set of option constraints (conflicts and dependencies passing the vote threshold of 5)
- Per-program constraint table (constraints_per_program.csv) feeding high-risk combination prediction
- Precision/recall statistics (overall and per-program) comparable to the reported 94.00% / 92.48% / 80.56%

## How to apply

Build the candidate pool with extractRelationships in llm_interface/constraint.py, which queries GPT-4 Turbo (gpt-4-1106-preview) at temperature 0.7 with n=10 inferences per call over option descriptions parsed from Groff man pages (program name via .TH, sections via .SH, options via .TP/.PP/.RS/.sp); the high temperature and repeated sampling maximize constraint diversity. Then run checkRelationships from the same file: each candidate conflict or dependency is re-submitted to the LLM under bidirectional reasoning — a verification question (does the constraint hold given the documented options?) plus a counterexample question — at temperature 0.2, since checking needs consistency rather than diversity. Tally the self-check votes per candidate and retain a constraint only when the voting threshold of 5 is reached; rejected candidates are treated as LLM hallucination. This matters because of Challenge C1: LLMs tend to flag combinations that violate constraints (conflicts and dependencies) as high-risk targets, so a wrong constraint would misdirect fuzzing toward safe combinations. Finally, aggregate surviving constraints into the per-program table (constraints_per_program.csv) and benchmark against the paper values — 633 constraints, 94.00% overall precision, 92.48% average per-program precision — and, on the 20-program annotated subset, recall against assets/dataset/groundtruth_for_20_programs. (180 ground-truth constraints, reported recall 80.56%).

## Related tools

- **ProphetFuzz** (Hosts checkRelationships (the bidirectional-reasoning self-check) and extractRelationships in llm_interface/constraint.py; checkRelationships filters candidates by the voting threshold) — github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (The LLM interrogated by the self-check: extraction at temperature 0.7 with n=10 calls, self-check verification/counterexample questions at temperature 0.2 (gpt-4-1106-preview via OpenAI API))
- **manpage_parser** (Parses Groff man pages (.TH/.SH/.TP/.PP/.RS/.sp) into the structured option descriptions that both candidate extraction and self-check questions are grounded in) — github.com/NASP-THU/ProphetFuzz

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Overall precision of retained constraints is near 94.00% and per-program average precision near 92.48% across the 52-program documentation set (paper benchmark).
- Total retained constraints across the 52 programs is close to the reported 633.
- On the 20-program annotated subset, recall against assets/dataset/groundtruth_for_20_programs. (180 ground-truth constraints) is comparable to the reported 80.56%.
- Vote-count invariant: every retained conflict/dependency has at least 5 affirmative self-check votes at temperature 0.2; candidates below the threshold of 5 must be absent from the output table.
- Schema check: constraints_per_program.csv contains only conflicts and dependencies whose option names appear in that program's parsed man page (no options invented outside the .TP/.PP/.RS/.sp descriptions).

## Limitations

- The filter is itself LLM-based, so hallucination is reduced rather than eliminated — about 6% of retained constraints are still wrong at the reported 94.00% overall precision.
- Recall trade-off: true constraints can be voted out, capping recall at the reported 80.56% on the 20-program ground truth.
- Thresholds (temperature 0.7/0.2, n=10, vote threshold 5) were tuned for GPT-4 Turbo (gpt-4-1106-preview); other models may shift precision/recall and need re-tuning.
- Repeated self-check calls multiply API cost — the full ProphetFuzz pipeline averaged $8.69 per program.

## Evidence

- [methods] Bidirectional self-check inspects and filters incorrect constraint extractions: "Prophet Fuzzemploysabidirectionalreason- ing-based self-check method to inspect and filter out incorrect constraint extraction results"
- [intro] Rationale: wrong constraints cause the LLM to flag constraint-violating combinations as high-risk: [evidence span withheld — anchor status: not_found]
- [methods] Constraint extraction runs hot (0.7) for diversity, implying the self-check uses different settings: "0.7 for tasks needing diverse outcomes, like constraint extraction and command assembly"
- [readme] Self-check is implemented in checkRelationships in llm_interface/constraint.py: "| 3.2 | Self Check | [llm_interface/constraint.py](llm_interface/constraint.py) |  checkRelationships |"
- [readme] Repository ships a ground-truth dataset for evaluating constraint extraction: "4. assets/dataset: Dataset for eveluating constraint extraction module."
