---
name: few-shot-prompt-construction
description: Use when you must elicit LLM predictions of risky option combinations
  (or similar structured risky configurations) from parsed Groff man pages and a constraint
  list, using few-shot reasoning examples.
when_to_use_negative:
- Input documentation is not Groff man pages or lacks parseable option descriptions
  — constraint extraction and risk prediction will be unreliable
- No self-checked constraint list is available — the LLM will tend to flag combinations
  that violate option conflicts/dependencies as high-risk (challenge C1)
- The goal is command assembly or fuzzing rather than predicting which option combinations
  are high-risk
tools:
- name: ProphetFuzz
  role: Tool whose few-shot_generate.py (generatePrompt) builds the Auto CoT few-shot
    prompt and whose predict.py (predictCombinations) consumes it
  repo: github.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM queried via the OpenAI API (temperature 0.7, n=10) with the few-shot CoT
    prompt to predict high-risk option combinations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:18de8f770061b945ab21b4af75b05de8b7ec2b219b0f64bea0827cea4f60e721
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# few-shot-prompt-construction

## Summary

Constructs few-shot chain-of-thought (Auto CoT) prompts that embed self-checked option constraints so an LLM can predict high-risk option combinations for a command-line program from parsed documentation, as done in ProphetFuzz Section 3.3.

## When to use

Use when you must elicit LLM predictions of risky option combinations (or similar structured risky configurations) from parsed Groff man pages and a constraint list, using few-shot reasoning examples.

## When NOT to use

- Input documentation is not Groff man pages or lacks parseable option descriptions — constraint extraction and risk prediction will be unreliable
- No self-checked constraint list is available — the LLM will tend to flag combinations that violate option conflicts/dependencies as high-risk (challenge C1)
- The goal is command assembly or fuzzing rather than predicting which option combinations are high-risk

## Inputs

- Parsed Groff man-page content per program (program name, description, synopsis, option descriptions)
- Self-checked constraint list (conflicts and dependencies between options) from the constraint extraction + self-check steps
- Auto CoT few-shot exemplar man-page JSON files (e.g. manpage_htmldoc., manpage_jbig2., manpage_makeswf., manpage_yasm.)

## Outputs

- Per-combination JSON records (program, option combination, rationale) written to predicted_combinations
- Deduplicated set of predicted high-risk option combinations per program (across 52 programs totaling 1748 predictions)

## How to apply

First parse each program's Groff man page into structured fields (program name, description, synopsis, option descriptions) and obtain the constraint list already filtered by bidirectional-reasoning self-check, so the prompt can steer the LLM away from flagging combinations that violate conflicts/dependencies. Build the Auto CoT few-shot examples once (generated with n=1) using the exemplar man-page JSON files shipped in llm_interface/few-shot/ (e.g. manpage_htmldoc., manpage_jhead., manpage_yasm.), and attach them to a multi-step chain-of-thought prediction prompt (llm_interface/few-shot_generate.py, function generatePrompt). Query GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 and parameter n=10 so ten diverse inferences are made per program, then deduplicate the ten inference outputs into a consolidated set of predicted high-risk option combinations. The rationale: constraints suppress hallucinated conflict-violating predictions, while few-shot CoT examples anchor the risk categories (e.g. resource management and limits, complex data processing) and output format.

## Related tools

- **ProphetFuzz** (Tool whose few-shot_generate.py (generatePrompt) builds the Auto CoT few-shot prompt and whose predict.py (predictCombinations) consumes it) — github.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM queried via the OpenAI API (temperature 0.7, n=10) with the few-shot CoT prompt to predict high-risk option combinations)

## Examples

```
python llm_interface/few-shot_generate.py  # build Auto CoT few-shot prompt via generatePrompt, then call predictCombinations in llm_interface/predict.py (GPT-4 Turbo, temperature 0.7, n=10)
```

## Evaluation signals

- Every per-combination record parses as well-formed JSON with the expected fields (program, option combination, rationale)
- Consolidated prediction count across the 52 programs reaches approximately the reported 1748 high-risk option combinations
- No predicted combination violates constraints identified in the self-checked conflict/dependency list
- Predictions map onto the documented high-risk categories (resource management and limits, complex data processing)
- Ten inferences per program (n=10) are produced and deduplicated before counting

## Limitations

- LLM hallucination threatens constraint-extraction precision, which directly degrades prediction quality; the bidirectional-reasoning self-check must run first
- Predictions are made with temperature 0.7, so outputs are stochastic and must be consolidated/deduplicated across n=10 inferences
- Few-shot exemplars cover only eight example programs (htmldoc, jbig2, jhead, makeswf, mp4box, opj_compress, pdf2swf, yasm); programs far outside these styles may transfer poorly
- Average cost was $8.69 per program, so scaling to very large program sets incurs nontrivial API cost

## Evidence
