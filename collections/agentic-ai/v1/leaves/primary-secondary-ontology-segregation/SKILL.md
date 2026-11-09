---
name: primary-secondary-ontology-segregation
description: Use when free-text radiology concepts have already been extracted and
  mapped to SNOMED-CT/RADLEX ontologies, and they must be split into principal pathologies
  versus modifiers (severity, location) before protocol categorization.
when_to_use_negative:
- The input is already a protocol-compliant structured report or an existing feature
  table of labeled concepts — no ontology segregation is needed.
- Concepts are already present in cached_vocab. with their protocol-specific categorization,
  so the filter step would redundantly reprocess cached terms.
- No ontology mappings exist (e.g., the BioPortal API key is missing or concepts map
  to no SNOMED-CT/RADLEX term), leaving nothing valid to split into Primary/Secondary
  classes.
tools:
- name: MedPAO
  role: Agent framework whose Plan-Act-Observe loop orchestrates the filter_ontology
    tool and evaluates its output
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Core LLM engine (port 8080) that performs the few-shot Primary/Secondary classification
    of ontology mappings
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned concept extractor (port 8081) that produces the concepts whose
    ontology mappings are segregated
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: BioPortal Annotator API
  role: Provides the SNOMED-CT and RADLEX mappings (with hierarchy) that are subsequently
    split into Primary and Secondary sets
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:f0094edfe38903d2d5ce5e01ad7b2300059dd85e633c9648b16f04c383ef1103
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
  - mcp-python-sdk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# primary-secondary-ontology-segregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MedPAO filter step that classifies the SNOMED-CT and RADLEX ontology mappings of each extracted medical concept into Primary (principal pathology) and Secondary (modifier) categories via reasoning-based few-shot LLM prompting, so downstream protocol categorization (e.g., ABCDEF/P.A.B.C.D.E.F) receives a structured ontology signal instead of a flat concept list.

## When to use

Use when free-text radiology concepts have already been extracted and mapped to SNOMED-CT/RADLEX ontologies, and they must be split into principal pathologies versus modifiers (severity, location) before protocol categorization.

## When NOT to use

- The input is already a protocol-compliant structured report or an existing feature table of labeled concepts — no ontology segregation is needed.
- Concepts are already present in cached_vocab. with their protocol-specific categorization, so the filter step would redundantly reprocess cached terms.
- No ontology mappings exist (e.g., the BioPortal API key is missing or concepts map to no SNOMED-CT/RADLEX term), leaving nothing valid to split into Primary/Secondary classes.

## Inputs

- Free-text MIMIC-CXR radiology report (findings/impressions)
- Extracted concepts with source sentences (concepts_output. from get_concept)
- Lemmatized concept-to-ontology mappings from SNOMED-CT and RADLEX with hierarchy (ontologyMapping.csv from ontology_mapping)
- Optional cached concept-category pairs (cached_vocab.)

## Outputs

- segregated.: PRIMARY and SECONDARY classification of mapped ontologies for each concept
- new_concepts_output.: concepts not found in the local cache, feeding the filtering step

## How to apply

First run get_concept with the fine-tuned Med Llama-8B extractor (port 8081) to obtain concepts and their source sentences, then lemmatize and map them to SNOMED-CT and RADLEX using the BioPortal Annotator API (ontology_mapping tool). Optionally run check_cache against cached_vocab. so recurring concepts already carry a categorization and are excluded from re-processing (the surviving concepts appear in new_concepts_output.). Then invoke filter_ontology (tools/filter_ontologies.py): the DeepSeek-R1-70B engine on port 8080 is prompted with few-shot examples to classify each concept's ontology mappings into Primary (principal pathology) and Secondary (modifiers such as severity or location). The rationale is that reasoning-based classification, rather than hand-crafted rules, gives the categorize_concepts step a clean separation between findings that map to protocol categories and qualifiers that modify them. Write the result to savings/segregated. in the savings/ directory for the agent's Observe step to evaluate.

## Related tools

- **MedPAO** (Agent framework whose Plan-Act-Observe loop orchestrates the filter_ontology tool and evaluates its output) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Core LLM engine (port 8080) that performs the few-shot Primary/Secondary classification of ontology mappings) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned concept extractor (port 8081) that produces the concepts whose ontology mappings are segregated) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **BioPortal Annotator API** (Provides the SNOMED-CT and RADLEX mappings (with hierarchy) that are subsequently split into Primary and Secondary sets)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- savings/segregated. exists, is valid JSON, and assigns every mapped concept a PRIMARY or SECONDARY label
- Each concept labeled Primary corresponds to a principal pathology from SNOMED-CT/RADLEX, while Secondary labels capture modifiers such as severity or location
- Concepts found in cached_vocab. are excluded from segregation and appear only in new_concepts_output.
- Downstream ABCDEF categorization of the segregated concepts reproduces reference mappings (e.g., 'clear lungs' under B) consistent with the reported F1-score of 0.96 on concept categorization
- The Observe step of the PAO loop accepts the filter output as adequately addressing the structuring goal without re-planning

## Limitations

- Segregation quality depends on few-shot LLM prompting; LLMs are prone to factual hallucination and lack verifiable reasoning, which is why MedPAO wraps this step in a verifiable tool rather than an end-to-end model call.
- Requires internet access and a valid BioPortal API key in apis.; without ontology mappings the filter step has no input.
- The step is demonstrated only on chest X-ray reports from MIMIC-CXR/Eye Gaze MIMIC-CXR with SNOMED-CT and RADLEX; other domains or ontologies are untested.

## Evidence
