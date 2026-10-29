---
name: llm-prompt-based-constraint-extraction
description: Use when you need to convert parsed CLI program documentation (option
  descriptions from Groff man pages) into a structured table of option conflicts/dependencies,
  and LLM hallucination must be controlled via a self-check.
when_to_use_negative:
- Option descriptions are not in Groff man-page format or lack per-option .TP-style
  description blocks, so the parser and LLM prompts lack the structured input they
  rely on
- A manually curated ground-truth constraint list already exists for the programs,
  making LLM extraction and self-check unnecessary
- The downstream task needs high-risk combination prediction, command assembly, or
  fuzzing rather than the constraint-extraction stage itself
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: ProphetFuzz
  role: Hosts extractRelationships and checkRelationships in llm_interface/constraint.py,
    plus manpage_parser for preprocessing
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM backend (gpt-4-1106-preview) invoked via OpenAI API at temperature 0.7
    with n=10 inferences for extraction and temperature 0.2 with voting threshold
    5 for self-check
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype, including the
    constraint-extraction and self-check pipeline
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:5ec57028c96be8d28b059ceaf5f326a42b5ba3a129ae96e4ddaffecd1ea52f3e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# llm-prompt-based-constraint-extraction

## Summary

Extracts inter-option constraints (conflicts and dependencies) from command-line program documentation by prompting GPT-4 Turbo with parsed Groff man-page content, then filters LLM hallucinations using a bidirectional-reasoning self-check (verification plus counterexample questions) with a voting threshold.

## When to use

Use when you need to convert parsed CLI program documentation (option descriptions from Groff man pages) into a structured table of option conflicts/dependencies, and LLM hallucination must be controlled via a self-check.

## When NOT to use

- Option descriptions are not in Groff man-page format or lack per-option .TP-style description blocks, so the parser and LLM prompts lack the structured input they rely on
- A manually curated ground-truth constraint list already exists for the programs, making LLM extraction and self-check unnecessary
- The downstream task needs high-risk combination prediction, command assembly, or fuzzing rather than the constraint-extraction stage itself

## Inputs

- Groff man pages of command-line programs (52-program dataset from 40 packages)
- Structured option descriptions produced by manpage_parser (program name, synopsis, option descriptions)
- OpenAI API key set in llm_interface/config/.env

## Outputs

- Per-program constraint table (constraints_per_program.csv) of conflicts and dependencies
- Filtered constraint set with precision measurable against assets/dataset/precision. and groundtruth_for_20_programs.

## How to apply

First parse Groff man pages (program name via .TH, sections via .SH, option descriptions via .TP/.PP/.RS/.sp) with the manpage_parser into structured option descriptions. Then call extractRelationships in llm_interface/constraint.py using GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 and n=10 inferences per call, since diverse-outcome tasks like constraint extraction use the higher temperature. Apply the bidirectional-reasoning self-check (checkRelationships) with temperature 0.2 and a self-check voting threshold of 5 to inspect and filter out incorrect constraints — verification plus counterexample questions guard against LLM hallucination. Aggregate surviving conflicts and dependencies into a per-program constraint table (constraints_per_program.csv). Judge correctness against the paper's benchmark: 633 constraints at 94.00% overall precision and 92.48% average per-program precision, and for the 20-program annotated subset, recall near the reported 80.56% against groundtruth_for_20_programs. (180 ground-truth constraints).

## Related tools

- **ProphetFuzz** (Hosts extractRelationships and checkRelationships in llm_interface/constraint.py, plus manpage_parser for preprocessing) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM backend (gpt-4-1106-preview) invoked via OpenAI API at temperature 0.7 with n=10 inferences for extraction and temperature 0.2 with voting threshold 5 for self-check)
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype, including the constraint-extraction and self-check pipeline)

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Overall extraction precision near the reported 94.00% across the 52-program documentation set (633 constraints total)
- Average per-program precision near the reported 92.48%
- On the 20-program annotated subset, recall comparable to the reported 80.56% against the 180 ground-truth constraints in groundtruth_for_20_programs.
- Constraints surviving self-check formed a coherent conflict/dependency table without hallucinated option pairs; outperforms Carpet Fuzz as the baseline comparison

## Limitations

- LLM hallucination threatens constraint-extraction precision, which is why the bidirectional self-check with voting threshold 5 is mandatory; skipping it degrades precision
- Results depend on GPT-4 Turbo (gpt-4-1106-preview) behavior; other LLMs may not reproduce the 94.00% precision or 80.56% recall
- The self-check does not guarantee perfect precision — some incorrect constraints can survive and pollute downstream high-risk combination prediction

## Evidence

- [intro] hallucination threat motivating self-check: [evidence span withheld — anchor status: not_found]
