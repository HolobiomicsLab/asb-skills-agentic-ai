---
name: security-testing-knowledge-distillation
description: Use when you have recorded LLM chain-of-thought outputs from high-risk
  option combination predictions and want to synthesize reusable, ranked security-testing
  knowledge categories from them.
when_to_use_negative:
- You only need raw vulnerability counts or fuzzing statistics — this skill distills
  qualitative knowledge from prediction traces, not fuzzing results
- The LLM outputs are raw prompts or constraint-extraction results rather than chain-of-thought
  prediction outputs from the high-risk combination prediction step
- The target programs lack documented option descriptions (e.g., no Groff man pages
  to ground the predictions)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3372
tools:
- name: GPT-4 Turbo
  role: LLM used both to summarize per-program knowledge of vulnerability-prone option-combination
    categories and to synthesize and rank the 15 knowledge items
- name: Python
  role: Used for text parsing to extract prediction-related content from the recorded
    CoT outputs
- name: ProphetFuzz
  role: Source of the chain-of-thought prediction outputs being distilled; its High-Risk
    Combination Prediction component (llm_interface/predict.py, predictCombinations)
    generates the traces
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:46bc1b78f79622fef9caf8406dd88c01cecdfb3a36f60af09ebcc0d483778684
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# security-testing-knowledge-distillation

## Summary

Distills LLM chain-of-thought prediction traces from ProphetFuzz's 1748 high-risk option combination predictions into a ranked table of 15 easy-to-understand knowledge items for software testers who do not understand security, each paired with supporting option-combination evidence.

## When to use

Use when you have recorded LLM chain-of-thought outputs from high-risk option combination predictions and want to synthesize reusable, ranked security-testing knowledge categories from them.

## When NOT to use

- You only need raw vulnerability counts or fuzzing statistics — this skill distills qualitative knowledge from prediction traces, not fuzzing results
- The LLM outputs are raw prompts or constraint-extraction results rather than chain-of-thought prediction outputs from the high-risk combination prediction step
- The target programs lack documented option descriptions (e.g., no Groff man pages to ground the predictions)

## Inputs

- Complete chain-of-thought (CoT) outputs recorded by the LLM during prediction runs covering the 1748 predicted high-risk option combinations across 52 evaluation programs
- Per-program option descriptions and prediction traces focused on the fourth step of the prediction prompt

## Outputs

- knowledge_table.csv containing exactly 15 ranked rows of (knowledge item, supporting option-combination evidence)
- Top-three categories expected: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation

## How to apply

Load the complete chain-of-thought (CoT) outputs recorded by the LLM (GPT-4 Turbo) during each prediction run — here, the 1748 predicted high-risk option combinations across 52 evaluation programs from 40 packages. Using Python-based text parsing, extract only content directly related to the predictions, focusing on the fourth step of the prediction prompt. For each program, instruct GPT-4 Turbo to 'summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities'. Compile the per-program summaries, have GPT-4 Turbo synthesize them, and rank the knowledge items from most to least common, setting the prompt context to 'For software testers who do not understand security, they hope to have easy-to-understand knowledge to conduct efficient security testing'. Write the ranked rows (knowledge item, supporting evidence) to a CSV table; the rationale is that distilled knowledge makes LLM-derived fuzzing intuition reusable for human testers and future tooling.

## Related tools

- **GPT-4 Turbo** (LLM used both to summarize per-program knowledge of vulnerability-prone option-combination categories and to synthesize and rank the 15 knowledge items)
- **Python** (Used for text parsing to extract prediction-related content from the recorded CoT outputs)
- **ProphetFuzz** (Source of the chain-of-thought prediction outputs being distilled; its High-Risk Combination Prediction component (llm_interface/predict.py, predictCombinations) generates the traces) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
python parse_cot_outputs.py --traces llm_interface/output/ --step 4 && python summarize_with_gpt4turbo.py --per-program --context 'For software testers who do not understand security' --output knowledge_table.csv
```

## Evaluation signals

- Verify knowledge_table.csv contains exactly 15 ranked rows
- Check the top three ranked knowledge items match the paper's reported categories: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation
- Confirm each knowledge row is paired with supporting option-combination evidence drawn from the prediction traces
- Confirm summaries were produced per-program (all 52 programs) before the final synthesis step, not in a single pass
- Check that extracted CoT content covers only prediction-related content (the fourth step of the prediction prompt), excluding constraint-extraction and command-assembly text

## Limitations

- The distilled knowledge is grounded in the 52-program evaluation dataset and may not generalize to program categories outside those 40 packages
- Ranking depends on GPT-4 Turbo summarization quality; LLM hallucination can affect which categories surface (the paper notes LLMs tend to identify constraint-violating combinations as high-risk, so prompt context matters)
- Requires access to the full recorded CoT outputs from the original prediction runs; without task_003-style traces the skill cannot be applied

## Evidence

- [supplementary] Resource Management and Limits. Options that affect re-source allocation and limits can lead to vulnerabilities: [evidence span withheld — anchor status: not_found]
- [supplementary] Complex Data Processing. Combinations of options that lead to complex data processing tasks can increase the risk of vulner-abilities: [evidence span withheld — anchor status: not_found]
