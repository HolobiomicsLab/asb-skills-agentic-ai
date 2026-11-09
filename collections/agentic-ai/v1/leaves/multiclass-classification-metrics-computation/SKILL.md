---
name: multiclass-classification-metrics-computation
description: Use when comparing predicted vs. radiologist-annotated concepts or predicted
  vs. ground-truth ABCDEF category labels over the 200 annotated MIMIC-CXR samples,
  and you need macro/weighted F1, Jaccard, Subset Accuracy, Hamming Loss, and McNemar
  p-values to benchmark systems like MedPAO against LLM.
when_to_use_negative:
- Evaluating free-text report generation quality (e.g., the clinician 4.52/5 ratings)
  rather than discrete concept or category predictions
- Input predictions are not aligned to radiologist ground-truth annotations, so no
  reference labels exist to compute classification metrics against
- The task is ontology mapping/filtering (PRIMARY vs SECONDARY via BioPortal Annotator
  API) rather than classification against A-F category labels
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: MedPAO
  role: Agent whose categorize_concepts predictions are evaluated with macro/weighted
    F1, Jaccard, and McNemar's test against baselines
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Med Llama-8B
  role: Fine-tuned concept extractor (LoRA, served via TGI on port 8081) whose extraction
    output is fuzzy-matched at 80%/90% thresholds to compute Table 1 metrics
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: Deepseek-R1-70B
  role: Baseline LLM run under identical extraction prompting to provide comparison
    rows for the metric tables
- name: Python
  role: Implements fuzzy matching, classification metric computation, confusion matrices,
    and CSV export (Python 3.8 or higher)
- name: mcp-python-sdk
  role: Dependency for running the MedPAO agent whose predictions feed the metric
    computation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:539b3b3141b7485e0cbe95be145f83923af2a03ebea3cc50d9b159b2056cc1ae
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
  - Med Llama-8B
  - MedPAO
  - Python
  - Qwen2.5-72B-Instruct
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multiclass-classification-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A metrics-computation skill for evaluating medical concept extraction and ABCDEF concept categorization as multiclass/multi-label classification, reporting macro- and weighted-averaged Precision, Recall, F1, plus Jaccard (IoU), Subset Accuracy, Hamming Loss, and McNemar's-test pairwise significance.

## When to use

Use when comparing predicted vs. radiologist-annotated concepts or predicted vs. ground-truth ABCDEF category labels over the 200 annotated MIMIC-CXR samples, and you need macro/weighted F1, Jaccard, Subset Accuracy, Hamming Loss, and McNemar p-values to benchmark systems like MedPAO against LLM baselines.

## When NOT to use

- Evaluating free-text report generation quality (e.g., the clinician 4.52/5 ratings) rather than discrete concept or category predictions
- Input predictions are not aligned to radiologist ground-truth annotations, so no reference labels exist to compute classification metrics against
- The task is ontology mapping/filtering (PRIMARY vs SECONDARY via BioPortal Annotator API) rather than classification against A-F category labels

## Inputs

- 200 radiologist-annotated MIMIC-CXR (Eye Gaze) reports with ground-truth extracted medical concepts
- Ground-truth ABCDEF protocol category labels and filtered PRIMARY/SECONDARY ontology assignments for each concept
- Predicted concepts with source sentences from the fine-tuned Med Llama-8B extractor and baseline LLMs (Deepseek-R1-70B, DeepSeek-R1-Distill-Llama-70B, DeepSeek-R1-Distill-Qwen-7B, Qwen2.5-7B)
- Predicted ABCDEF category assignments from MedPAO categorize_concepts and baseline LLM categorization runs (DeepSeek-R1-Distill-Llama-70B, Qwen2.5-72B-Instruct)

## Outputs

- concept_extraction_metrics.csv mirroring Table 1 (macro/weighted P, R, F1, Subset Accuracy, Hamming Loss at 80% and 90% fuzzy thresholds, McNemar p-values)
- categorization_metrics.csv mirroring Table 2 (macro/weighted F1 and Jaccard for MedPAO vs baselines)
- Per-model confusion matrices over the A-F categories (Figure 5)

## How to apply

Frame each system's predictions as classification against the Eye Gaze MIMIC-CXR ground-truth annotations (200 samples). For concept extraction, fuzzy-match predicted concepts to annotations at 80% and 90% similarity thresholds to absorb lexical variation (e.g., 'no effusion' vs 'absence of effusion'), then compute macro- and weighted-averaged Precision, Recall, F1, Subset Accuracy, and Hamming Loss at each threshold. For ABCDEF categorization, treat each concept's assigned A-F label as the class and compute per-class Precision, Recall, F1, and Jaccard Similarity (IoU), reporting both macro and weighted averages to handle class imbalance; also generate per-model confusion matrices over the six categories (as in Figure 5). Use McNemar's test for pairwise significance against the reference system (MedPAO), recording p-values. Verify against the paper's anchors: fine-tuned extractor F1 0.86 macro / 0.91 weighted @80% (0.80/0.84 @90%), Subset Accuracy 0.28, Hamming Loss 0.0023; categorization macro F1 0.94 / weighted F1 0.96 with Jaccard 0.92-0.94 vs DeepSeek-R1-Distill-Llama-70B 0.91/0.94 and Qwen2.5-72B-Instruct 0.92/0.95.

## Related tools

- **MedPAO** (Agent whose categorize_concepts predictions are evaluated with macro/weighted F1, Jaccard, and McNemar's test against baselines) — https://github.com/MiRL-IITM/medpao-agent
- **Med Llama-8B** (Fine-tuned concept extractor (LoRA, served via TGI on port 8081) whose extraction output is fuzzy-matched at 80%/90% thresholds to compute Table 1 metrics) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **Deepseek-R1-70B** (Baseline LLM run under identical extraction prompting to provide comparison rows for the metric tables)
- **Python** (Implements fuzzy matching, classification metric computation, confusion matrices, and CSV export (Python 3.8 or higher))
- **mcp-python-sdk** (Dependency for running the MedPAO agent whose predictions feed the metric computation)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"  # then compute metrics: python eval_metrics.py --ground-truth gt_annotations. --preds categorized_concepts. --fuzzy-thresholds 0.80 0.90 --out categorization_metrics.csv
```

## Evaluation signals

- Fine-tuned extractor row reproduces Table 1: F1 0.86 macro / 0.91 weighted @80% fuzzy confidence; 0.80 / 0.84 @90%; Subset Accuracy 0.28 and Hamming Loss 0.0023 @80%
- MedPAO categorization row reproduces Table 2: macro F1 0.94, weighted F1 0.96, Jaccard 0.92-0.94, meeting or exceeding DeepSeek-R1-Distill-Llama-70B (0.91/0.94) and Qwen2.5-72B-Instruct (0.92/0.95)
- Confusion matrices over A-F categories match Figure 5 per-model patterns
- McNemar's-test pairwise comparisons against MedPAO remain consistent with the reported significance results
- Output files concept_extraction_metrics.csv and categorization_metrics.csv are produced with both macro and weighted averages (class imbalance handled)

## Limitations

- Fuzzy thresholds of 80%/90% are needed to absorb lexical variation, so metrics are sensitive to threshold choice
- Class imbalance across the six ABCDEF categories requires both macro and weighted averaging; single-average reporting is insufficient
- Evaluation is limited to 200 annotated MIMIC-CXR samples and CXR report structuring; medical-image integration and real-time clinical deployment are not yet implemented
- Ground truth depends on the radiologist annotation set; disagreements in annotation propagate to all metrics

## Evidence

- [results] multiclass framing with fuzzy thresholds: [evidence span withheld — anchor status: not_found]
- [results] macro/weighted averages rationale: [evidence span withheld — anchor status: not_found]
- [results] Jaccard and confusion matrices for categorization: [evidence span withheld — anchor status: not_found]
