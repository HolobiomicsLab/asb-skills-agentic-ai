---
name: qualitative-knowledge-ranking-synthesis
description: Use when you have raw LLM CoT prediction outputs (e.g., 1748 predicted
  high-risk option combinations across 52 programs) and need to distill them into
  a small, ranked set of reusable security-testing knowledge items.
when_to_use_negative:
- The goal is quantitative validation of prediction accuracy (e.g., precision against
  ground truth) rather than qualitative knowledge distillation — use a precision-evaluation
  workflow instead
- You only need the raw high-risk combination predictions, not human-understandable
  categories
- The prediction traces lack per-program provenance, so per-program summaries cannot
  be compiled
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GPT-4 Turbo
  role: Per-program summarization of option-combination vulnerability knowledge and
    cross-program synthesis and ranking into 15 items
- name: Python
  role: Text parsing of CoT outputs to extract prediction-related content from the
    fourth step of the prediction prompt
- name: ProphetFuzz
  role: 'Source of the CoT prediction traces: predicts 1748 high-risk option combinations
    across 52 programs'
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:62f68106d94f3a48cf53d07d22358b4e1bb236f1edfb43a785ed7f2cc7ede08c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# qualitative-knowledge-ranking-synthesis

## Summary

Distills qualitative, security-testing knowledge from LLM chain-of-thought prediction traces by summarizing per-program option-combination insights with GPT-4 Turbo and synthesizing them into a ranked table of knowledge items with supporting evidence.

## When to use

Use when you have raw LLM CoT prediction outputs (e.g., 1748 predicted high-risk option combinations across 52 programs) and need to distill them into a small, ranked set of reusable security-testing knowledge items.

## When NOT to use

- The goal is quantitative validation of prediction accuracy (e.g., precision against ground truth) rather than qualitative knowledge distillation — use a precision-evaluation workflow instead
- You only need the raw high-risk combination predictions, not human-understandable categories
- The prediction traces lack per-program provenance, so per-program summaries cannot be compiled

## Inputs

- Complete chain-of-thought (CoT) outputs recorded by the LLM during each prediction run (1748 predicted high-risk option combinations across 52 evaluation programs)
- Prediction prompt definition (to isolate the fourth-step content)
- Per-program mapping of predictions to evaluation programs

## Outputs

- knowledge_table.csv with exactly 15 ranked rows of (knowledge item, supporting option-combination evidence)
- Ranked knowledge categories, expected top three: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation

## How to apply

Load the complete chain-of-thought (CoT) outputs recorded during each prediction run — here, the 1748 predicted high-risk option combinations across the 52 evaluation programs. Extract only the content directly related to the predictions, focusing on the fourth step of the prediction prompt, using Python-based text parsing. For each of the 52 programs, instruct GPT-4 Turbo to 'summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities'. Compile the per-program summaries and have GPT-4 Turbo synthesize them, ranking the knowledge items from most to least common; frame the prompt for the target audience ('For software testers who do not understand security, they hope to have easy-to-understand knowledge to conduct efficient security testing') so the items stay actionable, each paired with supporting option-combination evidence from the traces.

## Related tools

- **GPT-4 Turbo** (Per-program summarization of option-combination vulnerability knowledge and cross-program synthesis and ranking into 15 items)
- **Python** (Text parsing of CoT outputs to extract prediction-related content from the fourth step of the prediction prompt)
- **ProphetFuzz** (Source of the CoT prediction traces: predicts 1748 high-risk option combinations across 52 programs) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
python parse_cot.py --traces task_003/outputs --prompt-step 4 > predictions.txt; gpt4-turbo summarize --context 'For software testers who do not understand security...' --out knowledge_table.csv --rows 15
```

## Evaluation signals

- knowledge_table.csv contains exactly 15 ranked rows
- Each row pairs a knowledge item with supporting option-combination evidence drawn from the traces
- Top three ranked items match the paper's reported categories: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation
- Extracted text is limited to the prediction-related content of the fourth prompt step (no unrelated CoT noise)
- Per-program summaries exist for all 52 programs before synthesis

## Limitations

- The distilled knowledge depends on the LLM's summarization quality and the ranking is qualitative (by commonality), not statistically validated
- Knowledge is derived from a single LLM (GPT-4 Turbo) and a fixed 52-program evaluation dataset, so generality to other tools or models is unverified
- Only the fourth step of the prediction prompt is used; other CoT content is discarded and may contain useful signal

## Evidence

- [methods] Extract only the content directly related to the predictions, focusing on the fourth step of the prediction prompt, using Python-based text parsing: [evidence span withheld — anchor status: not_found]
- [methods] summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities: [evidence span withheld — anchor status: not_found]
- [methods] verify the table contains exactly 15 ranked rows and that the top three items match the paper's reported categories: [evidence span withheld — anchor status: not_found]
- [supplementary] Resource Management and Limits. Options that affect re-source allocation and limits can lead to vulnerabilities: [evidence span withheld — anchor status: not_found]
