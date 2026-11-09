---
name: abcdef-protocol-concept-categorization
description: Use when free-text MIMIC-CXR report findings have already been extracted
  as medical concepts with source sentences and ontologies, and they must be organized
  into the six ABCDEF protocol categories (A-F) for structured reporting.
when_to_use_negative:
- The target reporting protocol is not ABCDEF and the categorization prompt/protocol
  description has not been adapted — although the toolset is modular, this instantiation
  is hard-wired to the ABCDEF protocol description
- No concepts have been extracted and ontology-filtered yet — the tool expects concepts
  with filtered PRIMARY/SECONDARY ontologies as input, not raw free-text reports
- The goal is monolithic end-to-end report generation without verifiable intermediate
  steps — the rationale here is a transparent, protocol-grounded tool-based alternative
  to opaque LLM-only approaches
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_0219
tools:
- name: MedPAO
  role: Agent framework whose Plan-Act-Observe loop invokes categorize_concepts within
    the ABCDEF structuring pipeline
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Core LLM engine that receives the ABCDEF protocol description, concepts, and
    filtered ontologies via few-shot prompting to assign categories
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Qwen2.5-72B-Instruct
  role: Baseline LLM for the same categorization prompt, used for F1/Jaccard comparison
    in Table 2
- name: Python
  role: Implements the categorize_concepts tool and the metric computation (pandas/requests
    in the mcp-python-sdk based pipeline)
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:6c74317b0806b3ae7f0977b764a4143036da9883f9388b5c6aab309ee1ad9dab
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
  - BioPortal Annotator API
  - Deepseek-R1-70B
  - Med Llama-8B
  - MedPAO
  - Python
  - Qwen2.5-72B-Instruct
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# abcdef-protocol-concept-categorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign each extracted chest X-ray medical concept to one of the six A-F categories of the ABCDEF clinical protocol using LLM few-shot prompting (MedPAO's categorize_concepts tool), enabling protocol-compliant structured report generation. The paper reports macro/weighted F1 of 0.94/0.96 and Jaccard 0.92-0.94 for this sub-task.

## When to use

Use when free-text MIMIC-CXR report findings have already been extracted as medical concepts with source sentences and ontologies, and they must be organized into the six ABCDEF protocol categories (A-F) for structured reporting.

## When NOT to use

- The target reporting protocol is not ABCDEF and the categorization prompt/protocol description has not been adapted — although the toolset is modular, this instantiation is hard-wired to the ABCDEF protocol description
- No concepts have been extracted and ontology-filtered yet — the tool expects concepts with filtered PRIMARY/SECONDARY ontologies as input, not raw free-text reports
- The goal is monolithic end-to-end report generation without verifiable intermediate steps — the rationale here is a transparent, protocol-grounded tool-based alternative to opaque LLM-only approaches

## Inputs

- Medical concepts extracted from MIMIC-CXR free-text reports, each with its source sentence (output of get_concept, concepts_output.)
- Concepts mapped to SNOMED-CT and RADLEX ontologies and classified into PRIMARY/SECONDARY ontologies (output of filter_ontologies, segregated.)
- ABCDEF protocol description (full protocol text supplied in the few-shot prompt)
- cached_vocab. of previously categorized recurring concepts (optional, for check_cache)

## Outputs

- categorized_concepts.: each concept assigned to one of the six A-F ABCDEF protocol categories
- categorized_report.: final structured report with all six A-F keys populated (output of generate_report)
- categorization_metrics.csv: macro and weighted Precision, Recall, F1, and Jaccard (IoU) per system, plus per-category confusion matrices (for the evaluation variant)

## How to apply

Run MedPAO's categorize_concepts tool: prompt the LLM engine (DeepSeek-R1-Distill-Llama-70B served via TGI on port 8080) with the complete ABCDEF protocol description, the extracted concepts, and their filtered PRIMARY/SECONDARY ontologies, so the model assigns each concept to one of the six A-F categories. Optionally consult check_cache against cached_vocab. to reuse pre-categorized recurring concepts and skip ontology mapping/filtering. Frame the resulting assignments as a multiclass classification problem and compute per-category Precision, Recall, F1-score, and Jaccard similarity (IoU), reporting both macro and weighted averages to handle class imbalance, plus per-model confusion matrices over the A-F categories. Validate that MedPAO's macro F1 (0.94) and weighted F1 (0.96) with Jaccard 0.92-0.94 meet or exceed LLM-only baselines (DeepSeek-R1-Distill-Llama-70B 0.91/0.94 and Qwen2.5-72B-Instruct 0.92/0.95) as reported in Table 2. The categorized concepts and their source sentences then feed generate_report to produce the final ABCDEF structured report.

## Related tools

- **MedPAO** (Agent framework whose Plan-Act-Observe loop invokes categorize_concepts within the ABCDEF structuring pipeline) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Core LLM engine that receives the ABCDEF protocol description, concepts, and filtered ontologies via few-shot prompting to assign categories) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Qwen2.5-72B-Instruct** (Baseline LLM for the same categorization prompt, used for F1/Jaccard comparison in Table 2)
- **Python** (Implements the categorize_concepts tool and the metric computation (pandas/requests in the mcp-python-sdk based pipeline)) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, without using check_cache tool: {findings}"
```

## Evaluation signals

- categorized_concepts. is valid JSON and every concept is assigned to exactly one of the six A-F categories
- Macro F1 of 0.94 and weighted F1 of 0.96 with Jaccard 0.92-0.94 are reproduced, meeting or exceeding the DeepSeek-R1-Distill-Llama-70B (0.91/0.94) and Qwen2.5-72B-Instruct (0.92/0.95) baselines per Table 2
- Confusion matrices over the A-F categories match the pattern reported in Figure 5
- Reference mappings from Figure 4 are reproduced, e.g. 'clear lungs' and 'no pneumothorax' under B and 'normal heart size' under C
- categorized_report. contains all six keys A-F populated per the ABCDEF protocol

## Limitations

- The ABCDEF protocol instantiation is hard-wired to chest X-ray reporting; generalization to another protocol requires re-prompting with a different protocol description (shown in the paper's extension experiment)
- LLM-based categorization remains prone to hallucination without the protocol-grounded, verifiable tool structure; baseline LLM-only approaches perform measurably worse
- check_cache caching of recurring concepts is optional; without it, every concept incurs full categorization cost, and the cache only covers previously seen concepts

## Evidence

- [methods] MedPAO achieves an F1-score of 0.96 on the critical sub-task of concept categorization.: "MedPAO achieves an F1-score of 0.96 on the critical sub-task of concept categorization."
- [methods] Categorize the concepts into appropriate ABCDEF categories. (using categorize_concepts): "4. Categorize the concepts into appropriate ABCDEF
categories. (using categorize_concepts)"
- [methods] our framework caches recurring medical concepts and their protocol-specific categorizations: "our framework caches recurring medical con-
cents and their protocol-specific categorizations"
- [readme] categorized_concepts.: Contains output of categorize_concepts tool, i.e. Categorized concepts according to specified protocol.: "`categorized_concepts.`: Contains output of `categorize_concepts` tool, i.e., Categorized concepts according to specified protocol."
- [readme] Categorizes findings according to the P.A.B.C.D.E.F protocol: "- Categorizes findings according to the P.A.B.C.D.E.F protocol"
