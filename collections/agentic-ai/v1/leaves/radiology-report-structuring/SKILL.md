---
name: radiology-report-structuring
description: Use when the input is a free-text radiology (e.g., chest X-ray) report
  and the goal is a transparent, protocol-grounded structured report with findings
  organized under ABCDEF categories rather than opaque single-pass LLM output.
when_to_use_negative:
- The report is already structured or checklist-organized — the skill targets unstructured,
  free-text narratives where protocol structure has been lost in the dictated report.
- Multimodal input including medical images — image integration is proposed future
  work and not implemented.
- Real-time clinical deployment with strict latency requirements — current local TGI-hosted
  70B/8B models require optimization (model compression, hardware acceleration) not
  yet achieved.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Deepseek-R1-70B
  role: Core LLM engine (DeepSeek-R1-Distill-Llama-70B on TGI port 8080) performing
    Plan tool selection, ontology filtering, ABCDEF categorization, and Observe evaluation
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned concept extractor on TGI port 8081, used by get_concept with few-shot
    prompting to extract medical concepts and source sentences
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: BioPortal Annotator API
  role: Maps lemmatized concepts to SNOMED-CT and RADLEX ontologies in the ontology_mapping
    step
- name: mcp-python-sdk
  role: Implements the MCP client/server (mcp_client.py, mcp_server.py) that integrates
    the PAO loop with the medical processing tools
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python 3.8
  role: Runtime environment; dependencies installed via pip (mcp-python-sdk, pandas,
    requests, nltk, tqdm, huggingface_hub, regex)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:c4e01dfc3048cbff4ab3b741e526cb57254ba5705e58d70ca15adcd8a5e50ace
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
  - Python 3.8
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# radiology-report-structuring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A protocol-driven agent skill that structures free-text radiology reports (MIMIC-CXR chest X-ray narratives) into ABCDEF-protocol-compliant structured data via a Plan-Act-Observe loop over verifiable tools (concept extraction, SNOMED-CT/RADLEX mapping, ontology filtering, categorization).

## When to use

Use when the input is a free-text radiology (e.g., chest X-ray) report and the goal is a transparent, protocol-grounded structured report with findings organized under ABCDEF categories rather than opaque single-pass LLM output.

## When NOT to use

- The report is already structured or checklist-organized — the skill targets unstructured, free-text narratives where protocol structure has been lost in the dictated report.
- Multimodal input including medical images — image integration is proposed future work and not implemented.
- Real-time clinical deployment with strict latency requirements — current local TGI-hosted 70B/8B models require optimization (model compression, hardware acceleration) not yet achieved.

## Inputs

- Free-text chest X-ray report findings (e.g., MIMIC-CXR / Eye Gaze MIMIC-CXR reports, passed via the {findings} placeholder of the agent prompt)
- BioPortal API key stored in apis.
- cached_vocab. of previously processed concepts and their ABCDEF categories (optional, for check_cache)

## Outputs

- concepts_output. — extracted concepts and their source sentences (get_concept)
- new_concepts_output. — concepts not found in local cache (check_cache)
- ontologyMapping.csv — concepts with mapped ontologies and hierarchy (ontology_mapping)
- segregated. — PRIMARY/SECONDARY ontology classification per concept (filter_ontologies)
- categorized_concepts. — concepts categorized by ABCDEF protocol (categorize_concepts)
- categorized_report. — final structured report written to savings/

## How to apply

Run two local TGI servers — DeepSeek-R1-Distill-Llama-70B as the LLM engine on port 8080 and the fine-tuned Med Llama-8B concept extractor on port 8081 — and place a BioPortal API key in apis.. In the Plan step, the MCP client prompts the LLM engine with the available tools and prior executed steps to select the next tool; in the Act steps, run get_concept (few-shot concept extraction with source sentences), ontology_mapping via the BioPortal Annotator API to SNOMED-CT and RADLEX, filter_ontologies (few-shot classification of mappings into PRIMARY/SECONDARY), categorize_concepts (assign ABCDEF categories given the full protocol description), and generate_report. Optionally check_cache against cached_vocab. first to reuse prior categorizations of recurring concepts and skip ontology mapping/filtering. In the Observe step, the LLM evaluates whether tool output addresses the original query, iterating back to Plan if not; the final categorized_report. is written to savings/. Validation: the report must contain all six keys A–F populated per the ABCDEF protocol (e.g., 'clear lungs' and 'no pneumothorax' under B, 'normal heart size' under C), matching expert structure scores of 4.48–4.59 out of 5.

## Related tools

- **Deepseek-R1-70B** (Core LLM engine (DeepSeek-R1-Distill-Llama-70B on TGI port 8080) performing Plan tool selection, ontology filtering, ABCDEF categorization, and Observe evaluation) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned concept extractor on TGI port 8081, used by get_concept with few-shot prompting to extract medical concepts and source sentences) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **BioPortal Annotator API** (Maps lemmatized concepts to SNOMED-CT and RADLEX ontologies in the ontology_mapping step)
- **mcp-python-sdk** (Implements the MCP client/server (mcp_client.py, mcp_server.py) that integrates the PAO loop with the medical processing tools) — https://github.com/MiRL-IITM/medpao-agent
- **Python 3.8** (Runtime environment; dependencies installed via pip (mcp-python-sdk, pandas, requests, nltk, tqdm, huggingface_hub, regex))

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- categorized_report. contains all six ABCDEF keys (A–F) populated per the protocol
- Report reproduces the Figure 4 reference mapping, e.g., 'clear lungs' and 'no pneumothorax' under B and 'normal heart size' under C
- Concept categorization F1-score at or near the reported 0.96 on the categorization sub-task
- Expert structure scores in the reported 4.48–4.59 range (average 4.52/5) when outputs are clinician-rated
- Intermediate artifacts (concepts_output., ontologyMapping.csv, segregated., categorized_concepts.) are present in savings/ for each sample, indicating each PAO Act step executed

## Limitations

- LLM-only baselines are prone to hallucination and lack verifiable reasoning, which is why the tool-grounded loop exists — but the agent depends on BioPortal API availability and local TGI server uptime (ports 8080/8081).
- Integration with medical images is not yet implemented (proposed as future work).
- Real-time clinical deployment is not yet achieved and depends on model compression and hardware acceleration optimizations.
- check_cache (cached_vocab.) only accelerates inference for recurring concepts; new concepts still require full ontology mapping and filtering.

## Evidence
