---
name: man-page-option-combination-analysis
description: Use when the goal is to reconstruct or reuse ProphetFuzz's Auto-CoT few-shot
  corpus from man-page JSON files in llm_interface/few-shot/, or to analyze command-line
  option combinations for vulnerability risk using only documentation.
when_to_use_negative:
- The target program has no Groff/man-page documentation available, since the method
  is designed to operate 'with only documentation' and starts from keyword-matching
  extraction of program name, description, synopsis, and option descriptions.
- You need to fuzz already-predicted combinations or generate input files/commands
  — those are handled by downstream skills (Command Assembly, File Generation, Fuzzing),
  not this skill.
- The option combinations of interest already come from a labeled dataset rather than
  documentation, e.g. assets/dataset/groundtruth_for_20_programs. used for evaluating
  the constraint extraction module.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: ProphetFuzz
  role: Framework whose llm_interface/few-shot_generate.py (generatePrompt) and llm_interface/predict.py
    (predictCombinations) implement the Auto-CoT few-shot corpus generation and high-risk
    combination prediction from man-page JSON files
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM backend (gpt-4-1106-preview) queried via the OpenAI API to produce chain-of-thought
    analysis examples for historically vulnerable option combinations and to extract
    constraints
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype, including the
    llm_interface scripts for few-shot generation, constraint extraction, and prediction
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:6f70448d054f99d77221ae868478aedaec7e1294ea607bb209c0dc614a356203
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# man-page-option-combination-analysis

## Summary

A skill for analyzing Groff/man-page documentation of command-line programs to support LLM-driven prediction of high-risk option combinations, using an Auto-CoT-generated few-shot corpus (8 curated man-page programs, e.g. htmldoc, jbig2, jhead, makeswf, mp4box, opj_compress, pdf2swf, yasm).

## When to use

Use when the goal is to reconstruct or reuse ProphetFuzz's Auto-CoT few-shot corpus from man-page JSON files in llm_interface/few-shot/, or to analyze command-line option combinations for vulnerability risk using only documentation.

## When NOT to use

- The target program has no Groff/man-page documentation available, since the method is designed to operate 'with only documentation' and starts from keyword-matching extraction of program name, description, synopsis, and option descriptions.
- You need to fuzz already-predicted combinations or generate input files/commands — those are handled by downstream skills (Command Assembly, File Generation, Fuzzing), not this skill.
- The option combinations of interest already come from a labeled dataset rather than documentation, e.g. assets/dataset/groundtruth_for_20_programs. used for evaluating the constraint extraction module.

## Inputs

- Groff man-page documents (standard documentation format for command-line programs), parsed by manpage_parser/parser.py
- Man-page few-shot JSON files: llm_interface/few-shot/manpage_*. for 8 programs (htmldoc, jbig2, jhead, makeswf, mp4box, opj_compress, pdf2swf, yasm), each encoding historically vulnerable option combinations
- OpenAI API key set in llm_interface/config/.env

## Outputs

- few_shot_prompt JSON corpus containing LLM-generated chain-of-thought analysis examples, one per historically vulnerable option combination, covering the 8 curated man-page programs
- Predicted high-risk option combinations produced by predictCombinations in llm_interface/predict.py, guided by extracted and self-checked constraints

## How to apply

First load the man-page few-shot JSON files from llm_interface/few-shot/ of the ProphetFuzz repository (manpage_htmldoc., manpage_jbig2., manpage_jhead., manpage_makeswf., manpage_mp4box., manpage_opj_compress., manpage_pdf2swf., manpage_yasm.), each encoding historically vulnerable option combinations for one program. Invoke generatePrompt in llm_interface/few-shot_generate.py to build the Auto-CoT prompt template for each historically vulnerable option combination, and query GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with the generation parameter n set to 1 to obtain chain-of-thought analysis for each combination. Assemble the LLM-generated analysis examples across the 8 curated man-page programs into the few_shot_prompt JSON corpus, structured for consumption by predictCombinations in llm_interface/predict.py. Constraint extraction feeding this prediction should use temperature 0.7 for tasks needing diverse outcomes, and extracted constraints should be verified with the bidirectional-reasoning self-check (checkRelationships in llm_interface/constraint.py).

## Related tools

- **ProphetFuzz** (Framework whose llm_interface/few-shot_generate.py (generatePrompt) and llm_interface/predict.py (predictCombinations) implement the Auto-CoT few-shot corpus generation and high-risk combination prediction from man-page JSON files) — https://github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM backend (gpt-4-1106-preview) queried via the OpenAI API to produce chain-of-thought analysis examples for historically vulnerable option combinations and to extract constraints)
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype, including the llm_interface scripts for few-shot generation, constraint extraction, and prediction) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- few_shot_prompt file exists and parses as valid JSON.
- The few-shot corpus covers all 8 curated man-page programs (htmldoc, jbig2, jhead, makeswf, mp4box, opj_compress, pdf2swf, yasm).
- The corpus is structured for consumption by predictCombinations in llm_interface/predict.py during high-risk combination prediction.
- The OpenAI query uses n=1, since only one set of examples per historically vulnerable combination is needed.
- Constraint extraction results have passed the bidirectional-reasoning self-check (checkRelationships) before being used to guide prediction.

## Limitations

- The few-shot corpus is built from only 29 manually collected historical high-risk option combinations across 8 programs, so coverage of risk patterns is limited to those curated examples.
- LLMs tend to identify option combinations that merely violate constraints (conflicts and dependencies) as high-risk, so informing the LLM of correct constraints in advance is essential and constraint-extraction hallucination threatens precision.
- The prototype was tested on Ubuntu 20.04 in the provided Docker environment; outside Docker, dependencies must be installed manually and fuzzing_handler/config. must be adjusted with the program path.

## Evidence

- [supplementary] ProphetFuzz manually collects 29 historical high-risk option combinations across eight programs and employs an Auto CoT-based method to generate eight analysis examples as the few-shot (i.e., 8-shot) corpus.: [evidence span withheld — anchor status: not_found]
- [intro] ProphetFuzz parses Groff man pages and extracts program name, description, synopsis, and option descriptions by keyword matching before LLM analysis.: [evidence span withheld — anchor status: not_found]
