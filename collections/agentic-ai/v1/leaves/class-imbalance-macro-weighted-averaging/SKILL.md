---
name: class-imbalance-macro-weighted-averaging
description: Use when comparing classifiers (e.g., MedPAO vs. LLM baselines) on a
  multiclass labeling task with imbalanced category frequencies — such as ABCDEF protocol
  categories — where a single accuracy or pooled F1 would mask performance on minority
  classes.
when_to_use_negative:
- Binary classification or single-metric headline reporting where pooled accuracy
  already suffices and no meaningful class imbalance exists
- When gold labels for every concept are unavailable — macro/weighted F1 against partial
  annotations would be misleading
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_3474
tools:
- name: MedPAO
  role: Agent whose categorize_concepts tool produces the predicted ABCDEF category
    assignments being scored against ground truth
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Baseline LLM whose few-shot categorization predictions are scored with the
    same macro/weighted metrics for comparison
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Qwen2.5-72B-Instruct
  role: Second baseline LLM evaluated with identical metrics to establish the comparison
    in Table 2
- name: Python
  role: Computes Precision, Recall, F1, Jaccard (macro and weighted) and confusion
    matrices; writes categorization_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:a1b95800252be17926138c895f4edd5ec09759b1dec83d076b46818ac6a1e0f8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/MiRL-IITM/medpao-agent
  repo_url: https://github.com/MiRL-IITM/medpao-agent
  tools:
  - Deepseek-R1-70B
  - MedPAO
  - Python
  - Qwen2.5-72B-Instruct
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# class-imbalance-macro-weighted-averaging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates multiclass medical NLP outputs (e.g., ABCDEF concept categorization over radiologist-annotated MIMIC-CXR samples) by computing Precision, Recall, F1, and Jaccard (IoU) per class and reporting both macro and weighted averages, so rare protocol categories are not hidden by frequent ones.

## When to use

Use when comparing classifiers (e.g., MedPAO vs. LLM baselines) on a multiclass labeling task with imbalanced category frequencies — such as ABCDEF protocol categories — where a single accuracy or pooled F1 would mask performance on minority classes.

## When NOT to use

- Binary classification or single-metric headline reporting where pooled accuracy already suffices and no meaningful class imbalance exists
- When gold labels for every concept are unavailable — macro/weighted F1 against partial annotations would be misleading

## Inputs

- Ground-truth multiclass labels: ABCDEF protocol category per concept for 200 radiologist-annotated MIMIC-CXR samples
- Predicted category labels from each system being compared (e.g., MedPAO categorize_concepts output, DeepSeek-R1-Distill-Llama-70B and Qwen2.5-72B-Instruct few-shot predictions)

## Outputs

- Per-class and averaged Precision, Recall, F1-score, and Jaccard Similarity (IoU) with macro and weighted averages per system
- Per-model confusion matrices over the six A-F categories
- categorization_metrics.csv containing all computed metrics

## How to apply

Frame the task as multiclass classification: each medical concept extracted from the 200 radiologist-annotated MIMIC-CXR ground-truth samples is assigned to one of the six ABCDEF protocol categories. For each system (MedPAO, DeepSeek-R1-Distill-Llama-70B, Qwen2.5-72B-Instruct), compute per-class Precision, Recall, F1-score, and Jaccard Similarity (IoU) against the annotated gold labels, then report both macro average (unweighted mean over classes, treating each category equally) and weighted average (weighted by class support) to handle class imbalance. Generate per-model confusion matrices over the A-F categories (as in Figure 5) to localize which rare categories degrade performance. Save all metrics to categorization_metrics.csv and compare against the reported values (MedPAO macro F1 0.94 / weighted F1 0.96, Jaccard 0.92-0.94; DeepSeek-R1-Distill-Llama-70B 0.91/0.94; Qwen2.5-72B-Instruct 0.92/0.95) as an acceptance check.

## Related tools

- **MedPAO** (Agent whose categorize_concepts tool produces the predicted ABCDEF category assignments being scored against ground truth) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Baseline LLM whose few-shot categorization predictions are scored with the same macro/weighted metrics for comparison) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Qwen2.5-72B-Instruct** (Second baseline LLM evaluated with identical metrics to establish the comparison in Table 2)
- **Python** (Computes Precision, Recall, F1, Jaccard (macro and weighted) and confusion matrices; writes categorization_metrics.csv)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"  # then score categorized_concepts. vs gold with macro/weighted F1
```

## Evaluation signals

- Macro F1 for MedPAO equals 0.94 and weighted F1 equals 0.96, with Jaccard between 0.92 and 0.94, matching Table 2
- Baseline values reproduce: DeepSeek-R1-Distill-Llama-70B at 0.91 macro / 0.94 weighted F1 and Qwen2.5-72B-Instruct at 0.92 macro / 0.95 weighted F1, with MedPAO meeting or exceeding both
- Pairwise comparisons against baselines remain consistent with the McNemar test-style significance analysis
- Confusion matrix dimensions are 6x6 over the A-F categories and row sums equal class supports in the 200-sample ground truth
- categorization_metrics.csv is produced and contains per-class plus macro and weighted averages for every evaluated system

## Limitations

- Metrics depend on the fixed 200 radiologist-annotated MIMIC-CXR ground-truth samples; results are not guaranteed to transfer to other datasets or protocols
- Class imbalance means macro averages can be dominated by poorly performing minority categories even when weighted F1 is high
- The concept categorization score is a sub-task metric (F1 0.96) and does not by itself certify end-to-end report structuring quality

## Evidence

- [methods] Macro/weighted F1 and Jaccard to handle class imbalance: [evidence span withheld — anchor status: not_found]
- [methods] MedPAO reported performance: [evidence span withheld — anchor status: not_found]
