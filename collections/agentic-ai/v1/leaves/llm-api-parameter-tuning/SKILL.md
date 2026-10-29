---
name: llm-api-parameter-tuning
description: Use when driving GPT-4 Turbo (gpt-4-1106-preview) calls in an automated
  pipeline and you must decide per-stage settings for temperature and the generation
  parameter n based on whether diverse outcomes or a single deterministic example
  set is needed.
when_to_use_negative:
- Tasks where a single deterministic LLM output suffices (e.g., Auto-CoT example generation),
  where n=10 wastes budget without benefit
- Tasks needing strict reproducibility rather than diverse outcomes — temperature
  0.7 is explicitly chosen for diversity, not stability
- Non-OpenAI backends, since the prototype's parameter choices are grounded in the
  GPT-4 Turbo (gpt-4-1106-preview) API
tools:
- name: GPT-4 Turbo
  role: Target LLM (gpt-4-1106-preview) whose generation parameters (temperature,
    n) are tuned per pipeline stage
- name: OpenAI API
  role: API through which GPT-4 Turbo is queried with stage-specific temperature and
    n settings
- name: ProphetFuzz
  role: Host pipeline (llm_interface/predict.py predictCombinations, llm_interface/few-shot_generate.py
    generatePrompt) that consumes the parameterized calls
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype issuing the
    parameterized API calls
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:c4fa00d307473387691c0b6d57647d47c237f7eb0de91fefedf9b80be5d7c94c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# llm-api-parameter-tuning

## Summary

Configure OpenAI API generation parameters (model, temperature, n) for ProphetFuzz's LLM pipeline stages, choosing temperature 0.7 and n=10 for diversity-hungry tasks like high-risk combination prediction versus n=1 for one-off few-shot example generation.

## When to use

Use when driving GPT-4 Turbo (gpt-4-1106-preview) calls in an automated pipeline and you must decide per-stage settings for temperature and the generation parameter n based on whether diverse outcomes or a single deterministic example set is needed.

## When NOT to use

- Tasks where a single deterministic LLM output suffices (e.g., Auto-CoT example generation), where n=10 wastes budget without benefit
- Tasks needing strict reproducibility rather than diverse outcomes — temperature 0.7 is explicitly chosen for diversity, not stability
- Non-OpenAI backends, since the prototype's parameter choices are grounded in the GPT-4 Turbo (gpt-4-1106-preview) API

## Inputs

- Parsed Groff man-page content (program name, description, synopsis, option descriptions) per program
- Self-checked option constraint lists (conflicts/dependencies) from the constraint extraction step
- Auto-CoT few-shot JSON corpus (llm_interface/few-shot/manpage_*. for 8 curated programs)
- OpenAI API key configured in llm_interface/config/.env

## Outputs

- Per-program consolidated, deduplicated set of predicted high-risk option combinations (1748 across 52 programs)
- Per-combination JSON records containing program, option combination, and rationale
- Serialized few_shot_prompt. containing LLM-generated chain-of-thought analysis examples

## How to apply

Set the model to gpt-4-1106-preview, the variant ProphetFuzz implements its prototype on, and supply the API key via the llm_interface/config/.env file (OPENAI_API_KEY). For stages requiring diverse outcomes — constraint extraction, command assembly, and especially high-risk option combination prediction — set temperature to 0.7 and set n=10 so the LLM makes ten inferences per program, then consolidate the ten inferences into a deduplicated set of predicted combinations. For Auto-CoT few-shot corpus generation, where only one set of analysis examples is needed from the 29 historical high-risk combinations, set n=1. The rationale is that diversity parameters trade cost against coverage: n=10 per program at ~$8.69 per program underpins the 1748 predicted combinations, while n=1 avoids unnecessary spend on the one-shot corpus.

## Related tools

- **GPT-4 Turbo** (Target LLM (gpt-4-1106-preview) whose generation parameters (temperature, n) are tuned per pipeline stage)
- **OpenAI API** (API through which GPT-4 Turbo is queried with stage-specific temperature and n settings)
- **ProphetFuzz** (Host pipeline (llm_interface/predict.py predictCombinations, llm_interface/few-shot_generate.py generatePrompt) that consumes the parameterized calls) — https://github.com/NASP-THU/ProphetFuzz
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype issuing the parameterized API calls) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
openai.chat.completions.create(model="gpt-4-1106-preview", temperature=0.7, n=10, messages=predict_prompt)  # n=1 for few-shot_generate.py
```

## Evaluation signals

- Per-combination JSON records parse as well-formed JSON with the expected fields (program, option combination, rationale)
- Consolidated prediction count across the 52 programs reaches approximately the reported 1748 high-risk combinations
- few_shot_prompt. exists, parses as JSON, and covers the 8 curated man-page programs (htmldoc, jbig2, jhead, makeswf, mp4box, opj_compress, pdf2swf, yasm)
- API cost stays near the reported ~$8.69 per program, indicating n=10 is not being over- or under-issued
- Constraint-violating combinations are excluded, confirming the embedded constraints steered the diverse (temperature 0.7, n=10) inferences

## Limitations

- Parameter settings are tuned for gpt-4-1106-preview; other models may behave differently at temperature 0.7 with n=10
- LLM hallucination threatens constraint-extraction precision, so even well-tuned diverse sampling can flag combinations that violate real constraints
- Cost scales with n; n=10 inferences per program are needed to hit reported prediction counts but inflate API spend

## Evidence
