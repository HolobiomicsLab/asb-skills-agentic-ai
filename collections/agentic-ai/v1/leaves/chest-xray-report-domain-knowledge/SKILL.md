---
name: chest-xray-report-domain-knowledge
description: Use when free-text chest X-ray reports (e.g. from MIMIC-CXR) must be
  decomposed into protocol-compliant structured sections, or when extracted findings
  must be categorized into ABCDEF categories using SNOMED-CT/RADLEX vocabulary.
when_to_use_negative:
- The input is not a chest X-ray narrative report — the ABCDEF protocol mapping is
  CXR-specific and does not generalize to other clinical narrative types without re-deriving
  the protocol
- The task requires integration with medical images, which MedPAO does not yet implement
  (proposed as future work)
- Real-time clinical deployment is required, since the framework still depends on
  model compression and hardware acceleration optimizations
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3344
tools:
- name: MedPAO
  role: Protocol-driven agent that decomposes report structuring into a Plan-Act-Observe
    loop over specialized tools grounded in the ABCDEF protocol
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Core LLM engine of the agent used for reasoning and tool orchestration during
    protocol-based report structuring
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned (LoRA) medical concept recognition model providing get_concept
    extraction of concepts and source sentences
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: BioPortal Annotator API
  role: Maps extracted concepts to SNOMED-CT and RADLEX ontologies for ontology_mapping
    and filter_ontology steps
- name: mcp-python-sdk
  role: Model Context Protocol integration that lets the PAO loop dynamically orchestrate
    the medical processing tools
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:e8086ac58376dff8e7d5b1a84999a5daa3e1d8b487dc6686ff5a9c93efc45ea3
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chest-xray-report-domain-knowledge

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Domain knowledge for interpreting and structuring free-text chest X-ray radiology reports according to the ABCDEF (P.A.B.C.D.E.F) clinical protocol, with concept vocabulary grounded in SNOMED-CT and RADLEX ontologies, as instantiated by the MedPAO framework.

## When to use

Use when free-text chest X-ray reports (e.g. from MIMIC-CXR) must be decomposed into protocol-compliant structured sections, or when extracted findings must be categorized into ABCDEF categories using SNOMED-CT/RADLEX vocabulary.

## When NOT to use

- The input is not a chest X-ray narrative report — the ABCDEF protocol mapping is CXR-specific and does not generalize to other clinical narrative types without re-deriving the protocol
- The task requires integration with medical images, which MedPAO does not yet implement (proposed as future work)
- Real-time clinical deployment is required, since the framework still depends on model compression and hardware acceleration optimizations

## Inputs

- Free-text chest X-ray radiology reports (e.g. MIMIC-CXR findings text)
- Radiologist-annotated concept lists from the Eye Gaze MIMIC-CXR evaluation set (for validation)
- SNOMED-CT and RADLEX ontology mappings via BioPortal API
- cached_vocab.json of previously categorized concepts

## Outputs

- Concepts and their source sentences (concepts_output.json)
- Ontology mappings with hierarchy (ontologyMapping.csv)
- PRIMARY/SECONDARY ontology classification (segregated.json)
- ABCDEF-categorized concepts (categorized_concepts.json)
- Final protocol-compliant structured report (categorized_report.json)

## How to apply

Treat the narrative report as unstructured free text whose dictated form has lost the structured checklist organization, and map each extracted medical concept (via the get_concept tool, e.g. a LoRA fine-tuned Med Llama-8B model) back to the clinically established ABCDEF protocol for CXR analysis. Map extracted concepts to the SNOMED-CT and RADLEX ontologies with the BioPortal Annotator API, restrict to these predefined vocabularies, then classify mappings into PRIMARY and SECONDARY ontologies (filter_ontology). Route concepts into the correct ABCDEF category (categorize_concepts) — leveraging a cache of recurring concepts and their protocol-specific categorizations (check_cache / cached_vocab.json) — and assemble the final structured report (generate_report) under a Plan-Act-Observe loop with the Deepseek-R1-70B LLM engine. Judge correctness by concept categorization F1 (MedPAO reports 0.96 on this sub-task) and by expert radiologist/clinician ratings of the structured output (average 4.52/5 in the paper).

## Related tools

- **MedPAO** (Protocol-driven agent that decomposes report structuring into a Plan-Act-Observe loop over specialized tools grounded in the ABCDEF protocol) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Core LLM engine of the agent used for reasoning and tool orchestration during protocol-based report structuring) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned (LoRA) medical concept recognition model providing get_concept extraction of concepts and source sentences) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **BioPortal Annotator API** (Maps extracted concepts to SNOMED-CT and RADLEX ontologies for ontology_mapping and filter_ontology steps)
- **mcp-python-sdk** (Model Context Protocol integration that lets the PAO loop dynamically orchestrate the medical processing tools) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- Concept categorization F1-score reaches the reported 0.96 on the concept categorization sub-task
- Expert radiologist/clinician ratings of final structured outputs average around 4.52 out of 5
- Every extracted concept has a non-empty SNOMED-CT or RADLEX mapping and a PRIMARY/SECONDARY assignment in segregated.json
- categorized_concepts.json assigns each concept to exactly one ABCDEF category consistent with cached_vocab.json entries
- categorized_report.json contains all standardized P.A.B.C.D.E.F sections populated from the tool chain outputs

## Limitations

- Integration with medical images and automated suggestion systems is not yet implemented (future work)
- Real-time clinical deployment is not achieved; it depends on model compression and hardware acceleration optimizations
- Concepts are restricted to a predefined vocabulary from SNOMED-CT and RADLEX ontologies, so out-of-vocabulary findings may not be represented
- Internet access is required for BioPortal API calls, and local caching (cached_vocab.json) only covers previously seen concepts

## Evidence

- [intro] Demonstrate the protocol-driven approach by implementing the ABCDEF protocol, mapping narrative findings to a clinically established standard: "We demonstrate the efficacy of our protocol-driven approach by implementing the ABCDEF protocol, showing how our agent maps narrative findings to a clinically established standard."
- [methods] Ontology restriction to SNOMED-CT and RADLEX: "restricting them to a predefined vocabulary from SNOMEDCT and RADLEX ontologies"
- [methods] Caching of recurring concepts and protocol categorizations: [evidence span withheld — anchor status: not_found]
