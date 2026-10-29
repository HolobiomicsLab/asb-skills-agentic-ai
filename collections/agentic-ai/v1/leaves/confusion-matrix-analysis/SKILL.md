---
name: confusion-matrix-analysis
description: Use when evaluating multiclass agreement between predicted ABCDEF protocol
  category assignments (e.g., from MedPAO's categorize_concepts tool or LLM baselines)
  and radiologist ground truth on MIMIC-CXR samples, where macro/weighted F1 and Jaccard
  are the reporting targets.
when_to_use_negative:
- The input is raw free-text radiology reports that have not yet been processed by
  the concept extraction and categorization tools (get_concept, categorize_concepts)
  — no A–F predictions exist to score
- The evaluation goal is end-to-end report structuring quality (e.g., clinician ratings
  like the paper's 4.52/5) rather than the concept-categorization sub-task
- The task is ontology mapping/filtering (SNOMEDCT/RADLEX PRIMARY vs SECONDARY) rather
  than categorizing concepts into protocol categories
tools:
- name: MedPAO
  role: Agent whose categorize_concepts tool (few-shot prompting with the full ABCDEF
    protocol description plus concepts and filtered ontologies) produces the system-under-test
    A–F predictions
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: LLM engine powering MedPAO's categorize_concepts tool and serving as one baseline
    for the confusion-matrix comparison
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Qwen2.5-72B-Instruct
  role: Second baseline LLM run with the same categorization prompt to obtain predicted
    category assignments for the comparison
- name: Python
  role: Computes multiclass Precision, Recall, F1, Jaccard and the confusion matrices;
    writes categorization_metrics.csv
- name: mcp-python-sdk
  role: Implements the MCP client/server through which MedPAO's categorize_concepts
    tool is invoked to generate predictions
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:2bd0d2613961b84914fa1685b66b05f2c60c2ca5e7b6de724775b436f319f2c4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# confusion-matrix-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds per-model confusion matrices over the six ABCDEF protocol categories (A–F) to evaluate how well a system's concept categorization matches radiologist-annotated ground truth, summarizing Precision, Recall, F1, and Jaccard (IoU) in macro and weighted averages as done for MedPAO vs. LLM baselines.

## When to use

Use when evaluating multiclass agreement between predicted ABCDEF protocol category assignments (e.g., from MedPAO's categorize_concepts tool or LLM baselines) and radiologist ground truth on MIMIC-CXR samples, where macro/weighted F1 and Jaccard are the reporting targets.

## When NOT to use

- The input is raw free-text radiology reports that have not yet been processed by the concept extraction and categorization tools (get_concept, categorize_concepts) — no A–F predictions exist to score
- The evaluation goal is end-to-end report structuring quality (e.g., clinician ratings like the paper's 4.52/5) rather than the concept-categorization sub-task
- The task is ontology mapping/filtering (SNOMEDCT/RADLEX PRIMARY vs SECONDARY) rather than categorizing concepts into protocol categories

## Inputs

- 200 radiologist-annotated MIMIC-CXR ground-truth samples with medical concepts, filtered PRIMARY/SECONDARY ontologies, and ABCDEF category labels
- Predicted A–F category assignments from MedPAO (categorized_concepts. output of categorize_concepts tool)
- Predicted A–F category assignments from baseline LLMs (DeepSeek-R1-Distill-Llama-70B, Qwen2.5-72B-Instruct) via the same few-shot categorization prompt

## Outputs

- Per-model confusion matrices over the A–F ABCDEF categories
- categorization_metrics.csv with per-system macro and weighted Precision, Recall, F1-score, and Jaccard Similarity (IoU)

## How to apply

Load the 200 radiologist-annotated MIMIC-CXR ground-truth samples containing extracted medical concepts, their filtered PRIMARY/SECONDARY ontologies, and their ABCDEF protocol category labels. For each system (MedPAO via the categorize_concepts tool with few-shot prompting plus the full ABCDEF protocol description, and baselines such as DeepSeek-R1-Distill-Llama-70B and Qwen2.5-72B-Instruct), collect the predicted A–F label per concept from categorized_concepts.. Frame the comparison as multiclass classification over the six A–F categories and compute Precision, Recall, F1-score, and Jaccard Similarity (IoU) per class, reporting both macro and weighted averages to handle class imbalance. Generate the per-model confusion matrix over the A–F categories (as in the paper's Figure 5) and save metrics to categorization_metrics.csv.

## Related tools

- **MedPAO** (Agent whose categorize_concepts tool (few-shot prompting with the full ABCDEF protocol description plus concepts and filtered ontologies) produces the system-under-test A–F predictions) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (LLM engine powering MedPAO's categorize_concepts tool and serving as one baseline for the confusion-matrix comparison) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Qwen2.5-72B-Instruct** (Second baseline LLM run with the same categorization prompt to obtain predicted category assignments for the comparison)
- **Python** (Computes multiclass Precision, Recall, F1, Jaccard and the confusion matrices; writes categorization_metrics.csv)
- **mcp-python-sdk** (Implements the MCP client/server through which MedPAO's categorize_concepts tool is invoked to generate predictions) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, without using check_cache tool: {findings}"
```

## Evaluation signals

- MedPAO's macro F1 reaches 0.94 and weighted F1 reaches 0.96, with Jaccard between 0.92 and 0.94, meeting or exceeding DeepSeek-R1-Distill-Llama-70B (0.91/0.94) and Qwen2.5-72B-Instruct (0.92/0.95) as in Table 2
- The confusion matrix has exactly six classes corresponding to the A–F ABCDEF categories, matching Figure 5
- Both macro and weighted averages are reported so class imbalance is handled, and the weighted F1 of 0.96 for MedPAO matches the paper's reported headline F1
- categorization_metrics.csv is produced and pairwise comparisons against the baselines remain consistent under McNemar test-style pairwise comparison

## Limitations

- Metrics cover only the concept-categorization sub-task, not full report generation or the other MedPAO tools (get_concept, ontology_mapping, filter_ontology, generate_report)
- Ground truth is limited to 200 radiologist-annotated MIMIC-CXR samples, so class distribution and imbalance of A–F categories constrain the estimates
- Comparisons rely on the same few-shot prompt being applied identically to MedPAO and the baselines; protocol caching (check_cache/cached_vocab.) must be handled consistently between runs

## Evidence
