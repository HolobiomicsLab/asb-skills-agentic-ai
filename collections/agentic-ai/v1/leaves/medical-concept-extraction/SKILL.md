---
name: medical-concept-extraction
description: Use when the input is a free-text chest X-ray / radiology report and
  downstream ontology mapping or protocol categorization (e.g., ABCDEF) requires medically
  grounded concept spans plus their verbatim source sentences.
when_to_use_negative:
- The report text is already structured/checklist-formatted, so no concept extraction
  from narrative free text is needed
- The concepts are already present in the local cache (cached_vocab. via check_cache),
  which lets MedPAO skip extraction-adjacent ontology mapping and filtering for recurring
  concepts
- The task requires multimodal reasoning over the image itself — image integration
  is explicitly future work in MedPAO
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3384
- http://edamontology.org/topic_3444
tools:
- name: Med Llama-8B
  role: Fine-tuned (LoRA) medical concept recognition model executed with few-shot
    prompting via the get_concept tool
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: MedPAO
  role: Agent framework whose Plan-Act-Observe loop invokes concept extraction as
    the first Act step before ontology mapping and categorization
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: mcp-python-sdk
  role: Implements the MCP client/server that exposes get_concept as a verifiable
    tool
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:4071bde2554e8cb3d05a93152f5fc92a903036c6dce94349b6c8674d3a4cc554
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/MiRL-IITM/medpao-agent
  tools:
  - BioPortal Annotator API
  - Deepseek-R1-70B
  - Med Llama-8B
  - MedPAO
  - Python
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3384
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# medical-concept-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts medical concepts and their source sentences from free-text radiology reports using a LoRA fine-tuned Med Llama-8B model served via Huggingface TGI, with few-shot prompting. This is the first Act step of the MedPAO Plan-Act-Observe loop and outperforms LLM-based extraction baselines on MIMIC-CXR.

## When to use

Use when the input is a free-text chest X-ray / radiology report and downstream ontology mapping or protocol categorization (e.g., ABCDEF) requires medically grounded concept spans plus their verbatim source sentences.

## When NOT to use

- The report text is already structured/checklist-formatted, so no concept extraction from narrative free text is needed
- The concepts are already present in the local cache (cached_vocab. via check_cache), which lets MedPAO skip extraction-adjacent ontology mapping and filtering for recurring concepts
- The task requires multimodal reasoning over the image itself — image integration is explicitly future work in MedPAO

## Inputs

- Free-text medical/radiology report (e.g., findings section of a MIMIC-CXR chest X-ray report)
- Few-shot prompt with example report-to-concept mappings
- TGI server endpoint on port 8081 hosting shrishSVaidya/medllama_finetuned

## Outputs

- JSON list of extracted medical concepts, each paired with its source sentence (savings/concepts_output.)
- For evaluation: concept_extraction_metrics.csv with macro/weighted P/R/F1, Subset Accuracy, Hamming Loss at 80%/90% fuzzy thresholds, and McNemar p-values

## How to apply

Serve the fine-tuned model (shrishSVaidya/medllama_finetuned, Med Llama-8B, full precision, LoRA fine-tuned) via the Huggingface TGI framework on port 8081, while the DeepSeek-R1-Distill-Llama-70B reasoning engine runs on port 8080. Prompt the extractor with few-shot examples to extract medical concepts together with the sentences they came from, then lemmatize the raw concepts before mapping them to SNOMED-CT and RADLEX via the BioPortal Annotator API. Write the output to savings/concepts_output. in the MedPAO toolset. For evaluation, fuzzy-match predicted concepts against radiologist annotations at 80% and 90% similarity thresholds (to handle lexical variation like 'no effusion' vs 'absence of effusion') and compute macro/weighted Precision, Recall, F1, Subset Accuracy, and Hamming Loss as multi-label metrics. The few-shot design and fine-tuned weights exist because generic LLMs hallucinate concepts and fail the categorization sub-task that MedPAO's agent handles correctly.

## Related tools

- **Med Llama-8B** (Fine-tuned (LoRA) medical concept recognition model executed with few-shot prompting via the get_concept tool) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **MedPAO** (Agent framework whose Plan-Act-Observe loop invokes concept extraction as the first Act step before ontology mapping and categorization) — https://github.com/MiRL-IITM/medpao-agent
- **mcp-python-sdk** (Implements the MCP client/server that exposes get_concept as a verifiable tool) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, without using check_cache tool: {findings}"
```

## Evaluation signals

- Reproduced metrics for the fine-tuned extractor match Table 1: F1 0.86 macro / 0.91 weighted at the 80% fuzzy threshold and 0.80 / 0.84 at 90%, Subset Accuracy 0.28, Hamming Loss 0.0023 at 80%
- Every extracted concept is paired with a non-empty source sentence traceable to the original report
- Extraction output is valid JSON written to savings/concepts_output. and consumed downstream by ontology_mapping without parsing errors
- On the 200 ground-truth Eye Gaze MIMIC-CXR reports, McNemar's test against LLM baselines (Deepseek-R1-70B, DeepSeek-R1-Distill-Qwen-7B, Qwen2.5-7B, etc.) shows the fine-tuned tool is not significantly worse and baselines fail cases the extractor categorizes correctly

## Limitations

- Evaluation is fuzzy-matched at 80%/90% thresholds; lexical variation still affects Subset Accuracy (only 0.28 at 80%), so per-report perfect extraction is uncommon even though Hamming Loss is very low
- The extractor is trained/evaluated on chest X-ray reports (MIMIC-CXR, Eye Gaze subset); transfer to other clinical narrative domains is untested
- Requires a GPU/accelerator-served TGI endpoint; local LLM inference setup (e.g., Gaudi2 docker images) is needed rather than a hosted API

## Evidence

- [other] fuzzy matching rationale for lexical variation: [evidence span withheld — anchor status: not_found]
