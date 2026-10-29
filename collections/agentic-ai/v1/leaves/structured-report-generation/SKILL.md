---
name: structured-report-generation
description: Use when free-text clinical narratives (e.g., MIMIC-CXR radiology reports)
  must be converted into structured reports whose sections follow a named clinical
  protocol (ABCDEF), with verifiable tool-grounded steps instead of a single monolithic
  LLM call.
when_to_use_negative:
- The report is already structured (e.g., already a JSON/checklist document) — the
  skill targets unstructured, free-text narratives where checklist organization was
  lost in dictation
- The task requires integrating medical images or automated suggestion systems — the
  article lists these as future work, not implemented
- A real-time clinical deployment is required — the article notes this depends on
  model compression and hardware acceleration optimizations not yet achieved
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MedPAO
  role: Agent framework whose Plan-Act-Observe loop orchestrates the specialized tools
    (get_concept, ontology_mapping, filter_ontology, categorize_concepts, generate_report,
    check_cache) that produce the structured report
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: DeepSeek-R1-Distill-Llama-70B
  role: Core LLM engine (port 8080 via TGI) performing few-shot reasoning for ontology
    filtering and protocol categorization
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B (fine-tuned)
  role: Concept extraction model (port 8081 via TGI) that supplies the concepts and
    source sentences used to populate the structured report
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: BioPortal Annotator API
  role: Maps extracted concepts to SNOMED-CT and RADLEX vocabularies during the ontology_mapping
    step
  repo: https://bioportal.bioontology.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:72b93f9b44c350f4677e45b9f439d7c8d1722c583546b435eb977c7cc5548eb8
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
  - DeepSeek-R1-Distill-Llama-70B
  - Deepseek-R1-70B
  - Med Llama-8B
  - Med Llama-8B (fine-tuned)
  - MedPAO
  - Python
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# structured-report-generation-

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate protocol-compliant structured medical reports from free-text clinical narratives by decomposing the task into a Plan-Act-Observe loop over specialized, verifiable tools (concept extraction, ontology mapping/filtering, protocol categorization, report generation), as instantiated by MedPAO for the ABCDEF chest X-ray protocol on MIMIC-CXR reports.

## When to use

Use when free-text clinical narratives (e.g., MIMIC-CXR radiology reports) must be converted into structured reports whose sections follow a named clinical protocol (ABCDEF), with verifiable tool-grounded steps instead of a single monolithic LLM call.

## When NOT to use

- The report is already structured (e.g., already a JSON/checklist document) — the skill targets unstructured, free-text narratives where checklist organization was lost in dictation
- The task requires integrating medical images or automated suggestion systems — the article lists these as future work, not implemented
- A real-time clinical deployment is required — the article notes this depends on model compression and hardware acceleration optimizations not yet achieved

## Inputs

- Free-text radiology report (MIMIC-CXR findings narrative) passed via the {findings} prompt placeholder
- Clinical protocol definition (e.g., ABCDEF) with its section descriptions
- cached_vocab. local cache of previously processed concepts and their protocol categories (optional)
- BioPortal API key stored in apis.

## Outputs

- concepts_output. — extracted concepts and source sentences (get_concept)
- new_concepts_output. — concepts not found in local cache (check_cache)
- ontologyMapping.csv — concepts with mapped SNOMED-CT/RADLEX ontologies and hierarchy (ontology_mapping)
- segregated. — PRIMARY/SECONDARY classification of mapped ontologies (filter_ontologies)
- categorized_concepts. — concepts categorized per protocol section (categorize_concepts)
- categorized_report. — final structured report (generate_report)

## How to apply

Launch the MedPAO agent (mcp_client.py with mcp_server.py) with the DeepSeek-R1-70B LLM engine on port 8080 and the fine-tuned Med Llama-8B concept extractor on port 8081, and let the PAO loop sequence the tools. First run get_concept to extract medical concepts with their source sentences, then ontology_mapping to map lemmatized concepts to SNOMED-CT and RADLEX via the BioPortal Annotator API (API key in apis.). Use filter_ontology (few-shot LLM prompting) to classify mappings into Primary and Secondary ontologies, then categorize_concepts (few-shot prompting with the protocol description) to assign each concept to a protocol section; optionally consult check_cache against cached_vocab. for recurring concepts. Finally, generate_report assembles the fixed-JSON structured report from the categorized concepts and their original source sentences, writing categorized_report. to the savings/ directory. The rationale for decomposing into verifiable tools is to ground the LLM in a clinical protocol and avoid hallucination of a monolithic model.

## Related tools

- **MedPAO** (Agent framework whose Plan-Act-Observe loop orchestrates the specialized tools (get_concept, ontology_mapping, filter_ontology, categorize_concepts, generate_report, check_cache) that produce the structured report) — https://github.com/MiRL-IITM/medpao-agent
- **DeepSeek-R1-Distill-Llama-70B** (Core LLM engine (port 8080 via TGI) performing few-shot reasoning for ontology filtering and protocol categorization) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B (fine-tuned)** (Concept extraction model (port 8081 via TGI) that supplies the concepts and source sentences used to populate the structured report) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **BioPortal Annotator API** (Maps extracted concepts to SNOMED-CT and RADLEX vocabularies during the ontology_mapping step) — https://bioportal.bioontology.org/

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- categorized_report. exists in savings/, is valid JSON, and contains a distinct section for every category of the target protocol (e.g., ABCDEF)
- Concept categorization sub-task reaches the reported F1-score of 0.96
- Expert radiologists/clinicians rate the final structured outputs at the reported average of 4.52 out of 5 for clinical accuracy against MIMIC-CXR reference reports
- segregated. assigns each mapped ontology to exactly one of PRIMARY or SECONDARY for every extracted concept
- cached_vocab. accumulates unique concepts with their protocol categories and updates as the agent processes more samples

## Limitations

- Integration with medical images and automated suggestion systems is not implemented (proposed as future work)
- Real-time clinical deployment is not achieved; it depends on model compression and hardware acceleration optimizations
- Requires two local LLM inference servers (ports 8080 and 8080 for engine, 8081 for extractor) and internet access for BioPortal API calls, so it cannot run fully offline or on constrained hardware without those services

## Evidence
