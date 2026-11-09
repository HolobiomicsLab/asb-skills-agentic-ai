---
name: llm-chain-of-thought-prompting
description: Use when a target task requires an LLM to reason step-by-step over extracted
  documentation (e.g., option descriptions) and produce structured, deduplicated predictions,
  especially when embedded constraints must suppress invalid outputs.
when_to_use_negative:
- Predicted combinations that violate extracted constraints (conflicts/dependencies)
  — these should be filtered out by embedding the self-checked constraints in the
  prompt
- Targets without parsed documentation (e.g., binary-only programs with no Groff man
  pages), since the method is designed to work 'with only documentation'
- Tasks requiring deterministic, reproducible single-shot answers — the method uses
  temperature 0.7 and n=10 to elicit diverse outcomes
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: GPT-4 Turbo
  role: The LLM (gpt-4-1106-preview) queried via the OpenAI API with temperature 0.7
    and n=10 to execute the CoT prediction prompt per program
- name: ProphetFuzz
  role: Implements the CoT pipeline; High-Risk Combination Prediction lives in llm_interface/predict.py,
    predictCombinations; AutoCoT few-shot prompt generation in llm_interface/few-shot_generate.py,
    generatePrompt
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Language (~3k lines) in which the prototype and its CoT prompting scripts
    (few-shot_generate.py, predict.py) are implemented
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:410b63ab0754aca96db8d4b5dc776d7f374d5a898d8d31c627db12bdb9f34014
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# llm-chain-of-thought-prompting

## Summary

A method for driving LLM predictions via multi-step chain-of-thought (CoT) prompt engineering, as used in ProphetFuzz to predict high-risk command-line option combinations from parsed Groff man-page documentation alone.

## When to use

Use when a target task requires an LLM to reason step-by-step over extracted documentation (e.g., option descriptions) and produce structured, deduplicated predictions, especially when embedded constraints must suppress invalid outputs.

## When NOT to use

- Predicted combinations that violate extracted constraints (conflicts/dependencies) — these should be filtered out by embedding the self-checked constraints in the prompt
- Targets without parsed documentation (e.g., binary-only programs with no Groff man pages), since the method is designed to work 'with only documentation'
- Tasks requiring deterministic, reproducible single-shot answers — the method uses temperature 0.7 and n=10 to elicit diverse outcomes

## Inputs

- Parsed Groff man-page content (program name, description, usage synopsis, option descriptions)
- Self-checked list of option constraints (conflicts and dependencies) from bidirectional-reasoning self-check
- Auto CoT few-shot examples generated once (n=1) as man-page JSON files

## Outputs

- Deduplicated set of predicted high-risk option combinations per program
- Per-combination JSON records with fields: program, option combination, rationale
- Full chain-of-thought LLM output traces for later knowledge extraction

## How to apply

Build a multi-step (six-step) CoT prediction prompt whose inputs are the program's parsed Groff man-page content (program name, description, synopsis, option descriptions) and the self-checked constraint list produced by bidirectional-reasoning self-check, so the LLM avoids flagging combinations that violate conflicts/dependencies. Attach Auto CoT few-shot examples generated once (with n=1) from man-page JSON files (e.g., manpage_jhead., manpage_mp4box.) to guide reasoning format. Query GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 for diverse outcomes and parameter n=10 so the model makes ten inferences per program, then consolidate the ten inferences into a deduplicated set of predicted high-risk option combinations. Write per-combination JSON records (program, option combination, rationale) and retain the full CoT traces for downstream analysis.

## Related tools

- **GPT-4 Turbo** (The LLM (gpt-4-1106-preview) queried via the OpenAI API with temperature 0.7 and n=10 to execute the CoT prediction prompt per program)
- **ProphetFuzz** (Implements the CoT pipeline; High-Risk Combination Prediction lives in llm_interface/predict.py, predictCombinations; AutoCoT few-shot prompt generation in llm_interface/few-shot_generate.py, generatePrompt) — https://github.com/NASP-THU/ProphetFuzz
- **Python** (Language (~3k lines) in which the prototype and its CoT prompting scripts (few-shot_generate.py, predict.py) are implemented) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
bash run_all_in_one.sh bison  # runs constraint extraction, CoT few-shot generation, prediction, and fuzzing for one program
```

## Evaluation signals

- Every per-combination record parses as well-formed JSON with the expected fields (program, option combination, rationale)
- Consolidated prediction count across the 52 evaluation programs reaches approximately the reported 1748 high-risk option combinations
- Each program's ten inferences (n=10) are deduplicated into a single consolidated combination set
- A fraction (~12.30%) of predicted combinations is later validated by fuzzing, matching the paper's association with 364 unique vulnerabilities in 72 hours
- Downstream CoT-trace analysis distills exactly 15 ranked knowledge items with top three: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation

## Limitations

- LLM hallucination threatens constraint-extraction precision: LLMs tend to identify option combinations that violate constraints (conflicts and dependencies) as high-risk targets, so constraints must be embedded in the prompt in advance (challenge C1)
- Non-trivial API cost — the paper reports an average cost of only $8.69 per program for the full prediction over 52 programs
- Requires an OpenAI API key configured in llm_interface/config/.env; results vary due to temperature 0.7 sampling
- CoT outputs must be parsed programmatically (e.g., extracting the fourth prompt step) before they can be analyzed or summarized

## Evidence

- [intro] Hallucination failure mode motivating constraint embedding: [evidence span withheld — anchor status: not_found]
