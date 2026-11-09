---
name: few-shot-llm-prompting-for-medical-concepts
description: Use when free-text radiology concepts extracted from reports already
  have filtered PRIMARY/SECONDARY ontology mappings and must be assigned to one of
  six ABCDEF protocol categories using an LLM few-shot prompt.
when_to_use_negative:
- Input concepts have not yet been mapped and filtered into PRIMARY/SECONDARY SNOMED-CT/RADLEX
  ontologies (run ontology_mapping and filter_ontology first)
- The task is end-to-end report structuring or concept extraction from raw free text
  rather than categorizing already-extracted concepts
- The target protocol is not ABCDEF and no protocol description can be supplied in
  the few-shot prompt
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MedPAO
  role: Agent whose categorize_concepts tool performs the few-shot-prompted ABCDEF
    categorization within the PAO loop
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Core LLM engine receiving the few-shot categorization prompt (DeepSeek-R1-Distill-Llama-70B
    via TGI on port 8080)
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Qwen2.5-72B-Instruct
  role: Baseline LLM run with the same categorization prompt for comparison of predicted
    category assignments
- name: mcp-python-sdk
  role: MCP server/client plumbing that exposes categorize_concepts and check_cache
    as tools orchestrated by the PAO loop
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:53bdc34e0ebe12da0acfbd2bb69c7350b3db80814d192df504fbcc3db138dfc6
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# few-shot-llm-prompting-for-medical-concepts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns extracted radiology concepts (with their PRIMARY/SECONDARY SNOMED-CT and RADLEX ontology mappings) to the six ABCDEF protocol categories via few-shot prompting of an LLM engine, as implemented in MedPAO's categorize_concepts tool and evaluated with macro/weighted F1 and Jaccard metrics.

## When to use

Use when free-text radiology concepts extracted from reports already have filtered PRIMARY/SECONDARY ontology mappings and must be assigned to one of six ABCDEF protocol categories using an LLM few-shot prompt.

## When NOT to use

- Input concepts have not yet been mapped and filtered into PRIMARY/SECONDARY SNOMED-CT/RADLEX ontologies (run ontology_mapping and filter_ontology first)
- The task is end-to-end report structuring or concept extraction from raw free text rather than categorizing already-extracted concepts
- The target protocol is not ABCDEF and no protocol description can be supplied in the few-shot prompt

## Inputs

- Extracted medical concepts and their source sentences from a report (concepts_output. from get_concept)
- Filtered PRIMARY/SECONDARY SNOMED-CT and RADLEX ontology mappings per concept (segregated. from filter_ontologies)
- Full ABCDEF protocol description embedded in the few-shot prompt
- cached_vocab. of previously categorized concepts (optional, for check_cache)
- 200 radiologist-annotated MIMIC-CXR ground-truth category labels for evaluation

## Outputs

- categorized_concepts. — each concept assigned to one of the six A-F protocol categories
- Per-model confusion matrices over the A-F categories
- categorization_metrics.csv with macro/weighted Precision, Recall, F1, and Jaccard per system

## How to apply

Run the categorize_concepts tool as step 4 of the MedPAO pipeline (after get_concept, ontology_mapping, and filter_ontology), prompting the LLM engine (DeepSeek-R1-Distill-Llama-70B) with the full ABCDEF protocol description plus the concept list and their filtered ontologies to assign each concept to exactly one of the A-F categories. Optionally invoke check_cache first so previously categorized concepts in cached_vocab. are excluded from re-inference, reducing latency. Frame the result as multiclass classification over A-F and evaluate against 200 radiologist-annotated MIMIC-CXR ground-truth samples using Precision, Recall, F1-score, and Jaccard similarity (IoU), reporting both macro and weighted averages to handle class imbalance. Use the same few-shot prompt on baseline LLMs (e.g., Qwen2.5-72B-Instruct, DeepSeek-R1-Distill-Llama-70B alone) for comparison; acceptance is MedPAO macro F1 0.94 / weighted F1 0.96 with Jaccard 0.92-0.94 meeting or exceeding baselines (0.91/0.94 and 0.92/0.95).

## Related tools

- **MedPAO** (Agent whose categorize_concepts tool performs the few-shot-prompted ABCDEF categorization within the PAO loop) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Core LLM engine receiving the few-shot categorization prompt (DeepSeek-R1-Distill-Llama-70B via TGI on port 8080)) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Qwen2.5-72B-Instruct** (Baseline LLM run with the same categorization prompt for comparison of predicted category assignments)
- **mcp-python-sdk** (MCP server/client plumbing that exposes categorize_concepts and check_cache as tools orchestrated by the PAO loop) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- Macro F1 of 0.94 and weighted F1 of 0.96 on the 200 radiologist-annotated MIMIC-CXR samples meets or exceeds baselines (DeepSeek-R1-Distill-Llama-70B 0.91/0.94; Qwen2.5-72B-Instruct 0.92/0.95)
- Jaccard similarity (IoU) falls in the reported 0.92-0.94 range for MedPAO
- Per-category confusion matrices over A-F reproduce the Figure 5 comparison across MedPAO and baseline LLMs
- Every extracted concept receives exactly one valid A-F label; no concept is left uncategorized
- Pairwise (McNemar-style) comparison against baselines remains consistent with the reported Table 2 results
- Output written to categorized_concepts. matches the schema expected by generate_report

## Limitations

- Accuracy depends on upstream get_concept and filter_ontology outputs; hallucinated or unmapped concepts degrade categorization
- LLM-only few-shot baselines are prone to hallucination and lack verifiable reasoning, which is the motivation for the protocol-grounded agent approach
- Categorization is specific to the ABCDEF CXR protocol; other protocols require a new prompt and validated category labels
- Requires a running LLM engine server (TGI on port 8080) and, optionally, BioPortal API access for upstream ontology steps

## Evidence
