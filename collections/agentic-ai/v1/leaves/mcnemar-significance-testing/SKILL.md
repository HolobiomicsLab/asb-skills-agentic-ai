---
name: mcnemar-significance-testing
description: Use when comparing two systems on the same set of 200 ground-truth MIMIC-CXR
  reports with per-report correct/incorrect outcomes (e.g., fuzzy-matched concept
  extraction results) and a significance test of the paired difference is required.
when_to_use_negative:
- When the two systems were evaluated on different, non-paired report sets — McNemar's
  test requires discordant paired outcomes on identical inputs
- When the outcome of interest is a continuous score (e.g., per-report F1) rather
  than per-report correct/incorrect classification outcomes
- When only a single system's metrics are available and no baseline comparison is
  planned
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Med Llama-8B
  role: Fine-tuned concept extractor whose results form one arm of each paired McNemar
    comparison; served full precision via TGI on port 8081
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: Deepseek-R1-70B
  role: Baseline LLM whose per-report outcomes are compared pairwise against the fine-tuned
    extractor with McNemar's test
- name: mcp-python-sdk
  role: Dependency of the MedPAO agent that produced the per-report concept-extraction
    outputs being significance-tested
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python
  role: Used to compute the 2x2 contingency tables, run McNemar's test, and record
    p-values into concept_extraction_metrics.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:1a06af441a6d87f1f708a04811bb3db431f9c3da9d6305028da902e6d728223a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/MiRL-IITM/medpao-agent
  tools:
  - Deepseek-R1-70B
  - Med Llama-8B
  - MedPAO
  - Python
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# mcnemar-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies McNemar's test to pairwise-compare the fine-tuned MedPAO concept extractor (Med Llama-8B) against LLM baselines on the 200-report MIMIC-CXR concept categorization task, recording p-values that substantiate whether observed F1/precision/recall gaps are statistically significant.

## When to use

Use when comparing two systems on the same set of 200 ground-truth MIMIC-CXR reports with per-report correct/incorrect outcomes (e.g., fuzzy-matched concept extraction results) and a significance test of the paired difference is required.

## When NOT to use

- When the two systems were evaluated on different, non-paired report sets — McNemar's test requires discordant paired outcomes on identical inputs
- When the outcome of interest is a continuous score (e.g., per-report F1) rather than per-report correct/incorrect classification outcomes
- When only a single system's metrics are available and no baseline comparison is planned

## Inputs

- Per-report binary correct/incorrect outcomes for the fine-tuned Med Llama-8B concept extractor on 200 Eye Gaze MIMIC-CXR ground-truth reports
- Per-report binary outcomes for each baseline LLM (Deepseek-R1-70B, DeepSeek-R1-Distill-Llama-70B, DeepSeek-R1-Distill-Qwen-7B, Qwen2.5-7B) under identical extraction prompting
- Fuzzy-matched prediction labels at 80% and 90% similarity thresholds

## Outputs

- McNemar's test p-values for each pairwise comparison against the fine-tuned tool, at each fuzzy threshold
- Rows in concept_extraction_metrics.csv mirroring Table 1 with significance annotations

## How to apply

After running the fine-tuned Med Llama-8B concept extractor (LoRA fine-tuned, full precision, TGI on port 8081) and each baseline (Deepseek-R1-70B, DeepSeek-R1-Distill-Llama-70B, DeepSeek-R1-Distill-Qwen-7B, Qwen2.5-7B) under identical few-shot extraction prompting, convert each system's fuzzy-matched multi-label predictions (at 80% and 90% similarity thresholds) into per-report correct/incorrect binary outcomes. Build a 2x2 contingency table per baseline pair (baseline-correct/our-model-wrong vs. our-model-correct/baseline-wrong counts) and run McNemar's test to assess pairwise significance against the MedPAO fine-tuned tool, recording p-values alongside the macro/weighted F1, Subset Accuracy, and Hamming Loss metrics. The rationale is that McNemar's test is appropriate because both systems are evaluated on the same paired reports, so only discordant report outcomes inform the significance judgment. The reproduced fine-tuned-tool row should match Table 1 (F1 0.86 macro / 0.91 weighted @80% fuzzy confidence; 0.80 / 0.84 @90%; Subset Accuracy 0.28, Hamming Loss 0.0023 @80%) before significance is interpreted.

## Related tools

- **Med Llama-8B** (Fine-tuned concept extractor whose results form one arm of each paired McNemar comparison; served full precision via TGI on port 8081) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **Deepseek-R1-70B** (Baseline LLM whose per-report outcomes are compared pairwise against the fine-tuned extractor with McNemar's test)
- **mcp-python-sdk** (Dependency of the MedPAO agent that produced the per-report concept-extraction outputs being significance-tested) — https://github.com/MiRL-IITM/medpao-agent
- **Python** (Used to compute the 2x2 contingency tables, run McNemar's test, and record p-values into concept_extraction_metrics.csv)

## Examples

```
from statsmodels.stats.contingency_tables import mcnemar; res = mcnemar([[a,b],[c,d]], exact=False); print(res.pvalue)
```

## Evaluation signals

- The reproduced fine-tuned-tool row matches Table 1: F1 0.86 macro / 0.91 weighted @80% fuzzy confidence and 0.80 / 0.84 @90%, with Subset Accuracy 0.28 and Hamming Loss 0.0023 @80%
- A 2x2 contingency table exists per baseline pair with the two discordant cells (baseline-correct/ours-wrong, ours-correct/baseline-wrong) and a recorded McNemar p-value
- p-values are reported separately for the 80% and 90% fuzzy thresholds, showing threshold sensitivity of the significance conclusion
- concept_extraction_metrics.csv contains the pairwise significance columns mirroring the structure of Table 1

## Limitations

- McNemar's test is only valid on paired per-report outcomes from the same 200 ground-truth reports; unpaired re-sampling invalidates it
- Fuzzy thresholds (80% vs 90%) change the per-report outcome labels, so significance conclusions are threshold-dependent
- Hamming Loss 0.0023 and Subset Accuracy 0.28 indicate very sparse but frequent partial errors, which the binary correct/incorrect pairing may mask

## Evidence

- [methods] Assess pairwise significance against our model with McNemar's test and record p-values: [evidence span withheld — anchor status: not_found]
- [methods] Fuzzy-match predicted concepts against ground-truth annotations using similarity thresholds of 80% and 90%: [evidence span withheld — anchor status: not_found]
- [methods] Validation: the reproduced row for the fine-tuned tool should match Table 1: [evidence span withheld — anchor status: not_found]
