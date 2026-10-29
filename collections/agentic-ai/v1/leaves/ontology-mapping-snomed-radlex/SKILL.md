---
name: ontology-mapping-snomed-radlex
description: Use when raw medical concepts extracted from freestyle clinical reports
  (e.g., MIMIC-CXR text) must be normalized to a controlled vocabulary such as SNOMED-CT
  or RADLEX before protocol-based categorization.
when_to_use_negative:
- The input is already structured protocol-compliant data (e.g., an existing categorized_report.)
  — mapping would be redundant
- No network access or BioPortal API key is available, since the mapping depends on
  the BioPortal Annotator web service
- The target reporting domain uses ontologies outside the predefined SNOMED-CT/RADLEX
  vocabulary restriction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_3047
tools:
- name: BioPortal Annotator API
  role: Web service that annotates each lemmatized concept with SNOMED-CT and RADLEX
    ontology terms and hierarchies
- name: Med Llama-8B
  role: Fine-tuned concept extractor whose get_concept output provides the raw concepts
    fed into ontology mapping
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: ontology_mapping
  role: MedPAO tool (tools/ontology_mapping.py) that performs the lemmatization, BioPortal
    API calls, and writes ontologyMapping.csv
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: mcp-python-sdk
  role: SDK implementing the MCP server/client that exposes ontology_mapping as a
    verifiable tool orchestrated by the PAO loop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:d9fcb1ef64f5f411f215562d98ddebb5ff2dd7cc5f1509b7957089cfa0fc33d9
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
  - ontology_mapping
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ontology-mapping-snomed-radlex

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps free-text medical concepts extracted from radiology reports to standardized SNOMED-CT and RADLEX ontology terms using the BioPortal Annotator API, producing a concept-to-ontology lookup with hierarchy that downstream reasoning modules (filtering, protocol categorization) consume.

## When to use

Use when raw medical concepts extracted from freestyle clinical reports (e.g., MIMIC-CXR text) must be normalized to a controlled vocabulary such as SNOMED-CT or RADLEX before protocol-based categorization.

## When NOT to use

- The input is already structured protocol-compliant data (e.g., an existing categorized_report.) — mapping would be redundant
- No network access or BioPortal API key is available, since the mapping depends on the BioPortal Annotator web service
- The target reporting domain uses ontologies outside the predefined SNOMED-CT/RADLEX vocabulary restriction

## Inputs

- Raw medical concepts and their source sentences extracted from freestyle MIMIC-CXR reports (output of get_concept, concepts_output.)
- BioPortal API key stored in apis.
- Lemmatized concept strings (nltk preprocessing)

## Outputs

- ontologyMapping.csv: concepts with their mapped SNOMED-CT/RADLEX ontology terms and hierarchy
- Normalized ontology annotations consumed by filter_ontology and categorize_concepts steps

## How to apply

After the fine-tuned Med Llama-8B concept extractor returns concepts and their source sentences from a freestyle MIMIC-CXR report (get_concept), lemmatize each raw concept and submit it to the BioPortal Annotator API, restricting returned annotations to a predefined vocabulary of SNOMED-CT and RADLEX to promote interoperability. Record for each concept its mapped ontology term(s) and their hierarchy. Write the result as ontologyMapping.csv, which the filter_ontology step then splits into Primary (principal pathology) and Secondary (severity/location modifiers) classes via few-shot LLM prompting. Choosing SNOMED-CT and RADLEX is deliberate: the authors cite their comprehensive coverage and interoperability as the reason for using BioPortal Annotator over ad-hoc normalization.

## Related tools

- **BioPortal Annotator API** (Web service that annotates each lemmatized concept with SNOMED-CT and RADLEX ontology terms and hierarchies)
- **Med Llama-8B** (Fine-tuned concept extractor whose get_concept output provides the raw concepts fed into ontology mapping) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **ontology_mapping** (MedPAO tool (tools/ontology_mapping.py) that performs the lemmatization, BioPortal API calls, and writes ontologyMapping.csv) — https://github.com/MiRL-IITM/medpao-agent
- **mcp-python-sdk** (SDK implementing the MCP server/client that exposes ontology_mapping as a verifiable tool orchestrated by the PAO loop)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"  # ontology_mapping runs as step 2 within this PAO loop
```

## Evaluation signals

- ontologyMapping.csv exists in savings/ and contains, for every input concept, at least one mapped SNOMED-CT or RADLEX term plus its hierarchy
- Mapped terms are restricted to the predefined SNOMED-CT and RADLEX vocabularies (no annotations from other BioPortal ontologies leak through)
- Each mapped concept retains its link to the original source sentence so downstream categorization can ground the report
- Downstream filter_ontology output (segregated.) assigns every mapping to exactly one of PRIMARY or SECONDARY, implying mappings were complete enough to classify
- Final structured reports reach the paper's benchmark: F1 0.96 on concept categorization and mean radiologist accuracy 4.52/5

## Limitations

- Requires internet access for BioPortal API calls; no offline fallback is described in the README
- Mapping is restricted to SNOMED-CT and RADLEX, so concepts outside these vocabularies may remain unmapped
- LLM-based downstream classification of mappings into Primary/Secondary is few-shot prompting and may mislabel ambiguous modifiers
- Medical image integration and real-time clinical deployment are not yet implemented (future work per the discussion section)

## Evidence
