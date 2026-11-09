---
name: constraint-guided-option-reasoning
description: Use when you have parsed program documentation (name, description, synopsis,
  option descriptions) plus an extracted, self-checked constraint list and must predict
  which option combinations are high-risk vulnerability targets.
when_to_use_negative:
- Constraints between options have not yet been extracted and self-checked — prompting
  without them causes the LLM to flag combinations that violate conflicts/dependencies
  (challenge C1)
- The target is command assembly or file generation (Sections 3.4–3.5), not predicting
  which combinations are high-risk
- No parsed documentation exists (no Groff man pages to extract program name, description,
  synopsis, and option descriptions from)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: GPT-4 Turbo
  role: LLM (gpt-4-1106-preview) queried with temperature 0.7 and n=10 to predict
    high-risk option combinations from constraint-embedded CoT prompts
- name: ProphetFuzz
  role: Tool whose llm_interface/predict.py (predictCombinations) and llm_interface/few-shot_generate.py
    (AutoCoT generatePrompt) implement this prediction step
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: Python
  role: Implementation language for the prediction and few-shot generation scripts
    (about 3k lines of Python code)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:c0575e9562b7e50259fd8d108bcd6776241b2e029023954eddc648f6fb86002f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# constraint-guided-option-reasoning

## Summary

Predicts high-risk command-line option combinations from parsed Groff man-page documentation by prompting GPT-4 Turbo with self-checked option constraints (conflicts/dependencies) and AutoCoT few-shot examples, so the LLM avoids flagging combinations that violate constraints.

## When to use

Use when you have parsed program documentation (name, description, synopsis, option descriptions) plus an extracted, self-checked constraint list and must predict which option combinations are high-risk vulnerability targets.

## When NOT to use

- Constraints between options have not yet been extracted and self-checked — prompting without them causes the LLM to flag combinations that violate conflicts/dependencies (challenge C1)
- The target is command assembly or file generation (Sections 3.4–3.5), not predicting which combinations are high-risk
- No parsed documentation exists (no Groff man pages to extract program name, description, synopsis, and option descriptions from)

## Inputs

- Parsed Groff man-page content per program (program name, description, synopsis, option descriptions)
- Self-checked constraint list (conflicts and dependencies between options) from the constraint-extraction/self-check step
- AutoCoT few-shot examples generated once with n=1 (e.g., llm_interface/few-shot/manpage_*.)

## Outputs

- Deduplicated set of predicted high-risk option combinations per program
- Per-combination JSON records containing program, option combination, and rationale

## How to apply

Load each program's parsed Groff man-page content together with the constraint list produced by LLM constraint extraction and bidirectional-reasoning self-check (Section 3.2), because LLMs otherwise tend to flag combinations that violate conflicts/dependencies as high-risk. Build a multi-step chain-of-thought prediction prompt embedding these constraints, and attach AutoCoT few-shot examples generated once with n=1 from man-page pairs such as manpage_jhead. and manpage_yasm.. Query gpt-4-1106-preview via the OpenAI API with temperature 0.7 (for diverse outcomes) and parameter n=10 so the model makes ten inferences per program. Consolidate the ten inferences into a deduplicated set of predicted high-risk option combinations and write per-combination JSON records (program, option combination, rationale).

## Related tools

- **GPT-4 Turbo** (LLM (gpt-4-1106-preview) queried with temperature 0.7 and n=10 to predict high-risk option combinations from constraint-embedded CoT prompts)
- **ProphetFuzz** (Tool whose llm_interface/predict.py (predictCombinations) and llm_interface/few-shot_generate.py (AutoCoT generatePrompt) implement this prediction step) — https://github.com/NASP-THU/ProphetFuzz
- **Python** (Implementation language for the prediction and few-shot generation scripts (about 3k lines of Python code))

## Examples

```
bash run_all_in_one.sh bison  # runs constraint extraction, AutoCoT few-shot generation, high-risk combination prediction (llm_interface/predict.py), assembly, and fuzzing
```

## Evaluation signals

- Every per-combination record parses as well-formed JSON with the expected fields (program, option combination, rationale)
- Consolidated prediction count across the 52 programs reaches approximately the reported 1748 high-risk option combinations
- Predicted combinations do not violate extracted constraints (conflicts/dependencies) — the whole point of embedding constraints in the prompt
- Per-program cost is near the reported average of $8.69 when using gpt-4-1106-preview with temperature 0.7 and n=10
- Approximately 12.30% of predicted combinations are later associated with the 364 unique vulnerabilities found in 72 hours of fuzzing

## Limitations

- LLM hallucination threatens constraint-extraction precision, and uncorrected wrong constraints propagate into the prediction prompts (challenge C1)
- Predictions are probabilistic LLM outputs; temperature 0.7 and n=10 inferences introduce run-to-run variability that must be deduplicated
- Results depend on GPT-4 Turbo (gpt-4-1106-preview) as implemented in the prototype; other models are not validated in the paper

## Evidence

- [intro] Challenge C1 rationale: [evidence span withheld — anchor status: not_found]
- [intro] Constraint-guided prediction step: "Guided by the constraints identified, Prophet Fuzz predicts high-risk combinations based on the descriptions of the options."
- [methods] Self-check filters bad constraints before prediction: [evidence span withheld — anchor status: not_found]
