---
name: response-quality-mifr-scoring
description: Use when you have generated MLLM responses to jailbreak prompts (e.g.,
  from AdvBench, MMSafetyBench, or HarmBench runs) and need to quantify how well responses
  fulfill malicious intent, complementing binary ASR judgments.
when_to_use_negative:
- You only need binary attack success (bypass) counts — ASR judges such as GPT-4o
  Mini Judge, LlamaGuard3 Judge, or HarmBench Judge are the appropriate tools, not
  MIFR
- Responses were not generated from malicious queries with a defined attacker intent,
  so 'intent fulfillment' is undefined
- You are evaluating safety of the model itself (defense benchmarking) rather than
  attack outcome quality
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: quality_evaluate.py
  role: CLI entry point that computes MIFR quality metrics on generated responses
    via --input_file
  repo: https://github.com/thu-coai/JPS
- name: quality_judge
  role: Reasoning-LLM-based evaluator in the judge/ module that scores whether responses
    fulfill the attacker's malicious intent
  repo: https://github.com/thu-coai/JPS
- name: GPT-4o Mini Judge
  role: Safety judge used for ASR; its binary verdicts are contrasted with MIFR quality
    scores during aggregation
  repo: https://github.com/thu-coai/JPS
- name: HarmBench Judge
  role: Harm-classification judge for ASR evaluation on HarmBench responses, complemented
    by MIFR quality scoring
  repo: https://github.com/centerforaisafety/HarmBench
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:335c4aa54b40ab5b3075c520ba1d6714b51d776561d48737d61e3356d91ce65c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - CUDA
  - GPT-4o Mini Judge
  - HarmBench Judge
  - InternVL2-8B
  - MiniGPT4-13B
  - Python
  - Qwen2-VL-7B
  - quality_evaluate.py
  - quality_judge
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# response-quality-mifr-scoring

## Summary

Score jailbreak response quality using the Malicious Intent Fulfillment Rate (MIFR) metric, a Reasoning-LLM-based evaluator that measures whether generated responses actually fulfill the attacker's malicious intent, rather than only counting safety bypasses via ASR.

## When to use

Use when you have generated MLLM responses to jailbreak prompts (e.g., from AdvBench, MMSafetyBench, or HarmBench runs) and need to quantify how well responses fulfill malicious intent, complementing binary ASR judgments.

## When NOT to use

- You only need binary attack success (bypass) counts — ASR judges such as GPT-4o Mini Judge, LlamaGuard3 Judge, or HarmBench Judge are the appropriate tools, not MIFR
- Responses were not generated from malicious queries with a defined attacker intent, so 'intent fulfillment' is undefined
- You are evaluating safety of the model itself (defense benchmarking) rather than attack outcome quality

## Inputs

- Generated responses file from JPS inference phase (results/generate/responses) covering Qwen2-VL-7B, InternVL2-8B, and MiniGPT4-13B on AdvBench, MMSafetyBench, and HarmBench
- Original malicious query/prompt definitions from the benchmark datasets (AdvBench, MMSafetyBench, HarmBench) for intent-matching
- Reasoning-LLM-based quality judge configuration (quality_judge in judge/)

## Outputs

- Per-response MIFR quality scores indicating malicious intent fulfillment
- Aggregate MIFR rates per model per benchmark for inclusion in jps_asr_mifr_results.tsv
- Comparison of MIFR against ASR values to show response-quality gains over existing methods

## How to apply

After running the JPS attack and inference phases, collect the generated responses from results/generate/ and run the MIFR quality evaluation with quality_evaluate.py, passing the responses file via --input_file. The MIFR metric is assessed by a Reasoning-LLM-based evaluator (implemented as the quality judge) that examines whether the response content adheres to the attacker's intent and provides detailed, relevant outputs, going beyond simple refusal detection used by the GPT-4o Mini Judge, LlamaGuard3 Judge, and HarmBench Judge. Compute MIFR per model and per benchmark, then aggregate the values alongside ASR into the comparison table (e.g., jps_asr_mifr_results.tsv) so that quality improvements — the paper's reported 'Higher MIFR scores compared to existing methods' — can be verified across Qwen2-VL-7B, InternVL2-8B, and MiniGPT4-13B. Judge correct application by checking that every model-benchmark combination has both an ASR and a MIFR value and that MIFR reflects response quality, not merely bypass success.

## Related tools

- **quality_evaluate.py** (CLI entry point that computes MIFR quality metrics on generated responses via --input_file) — https://github.com/thu-coai/JPS
- **quality_judge** (Reasoning-LLM-based evaluator in the judge/ module that scores whether responses fulfill the attacker's malicious intent) — https://github.com/thu-coai/JPS
- **GPT-4o Mini Judge** (Safety judge used for ASR; its binary verdicts are contrasted with MIFR quality scores during aggregation) — https://github.com/thu-coai/JPS
- **HarmBench Judge** (Harm-classification judge for ASR evaluation on HarmBench responses, complemented by MIFR quality scoring) — https://github.com/centerforaisafety/HarmBench

## Examples

```
python quality_evaluate.py --input_file results/generate/responses
```

## Evaluation signals

- Every model-benchmark combination in the aggregated table (jps_asr_mifr_results.tsv) has both ASR and MIFR fields populated
- MIFR values reflect response quality/intent fulfillment, showing 'Higher MIFR scores compared to existing methods'
- MIFR results are 'Consistent results across different models and datasets' (Qwen2-VL-7B, InternVL2-8B, MiniGPT4-13B × AdvBench, MMSafetyBench, HarmBench)
- Responses file passed to quality_evaluate.py exists and is non-empty for each dataset config
- MIFR differs from ASR for the same responses, confirming the metric captures intent fulfillment rather than mere safety bypass

## Limitations

- MIFR depends on a Reasoning-LLM-based evaluator, so scores inherit the evaluator LLM's judgment biases and API availability constraints
- MIFR addresses the gap where 'Current research predominantly focuses on maximizing attack success rate (ASR), often overlooking whether the generated responses actually fulfill the attacker's malicious intent' — it is a complementary quality metric, not a replacement for ASR
- No discussion-section guidance is available in the extracted article (no changelog found), so edge-case handling of the evaluator is not documented

## Evidence

- [abstract] To evaluate the quality of attack outcomes, we propose the Malicious Intent Fulfillment Rate (MIFR) metric, assessed using a Reasoning-LLM-based evaluator: "To evaluate the quality of attack outcomes, we propose the Malicious Intent Fulfillment Rate (MIFR) metric, assessed using a Reasoning-LLM-based evaluator"
- [readme] quality_evaluate.py command: "python quality_evaluate.py --input_file results/generate/responses"
- [abstract] ASR vs MIFR framing: "Current research predominantly focuses on maximizing attack success rate (ASR), often overlooking whether the generated responses actually fulfill the attacker's malicious intent"
- [methods] Superior quality finding: "- **Superior Quality**: Higher MIFR scores compared to existing methods"
- [methods] Robust performance finding: "- **Robust Performance**: Consistent results across different models and datasets"
