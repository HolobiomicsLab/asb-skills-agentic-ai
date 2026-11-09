---
name: plan-act-observe-agent-orchestration
description: Use when a free-text narrative (e.g., a radiology report) must be transformed
  into protocol-compliant structured data and a transparent, tool-orchestrated agent
  is preferable to a monolithic LLM call prone to hallucination.
when_to_use_negative:
- The input is already structured or a checklist-organized report, so no free-text-to-protocol
  mapping is needed
- The task requires real-time clinical deployment or integration with medical images
  — both are explicitly unimplemented future work in the paper
- No internet access is available, since the ontology_mapping step depends on the
  BioPortal Annotator API
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3063
tools:
- name: MedPAO
  role: The agentic framework implementing the Plan-Act-Observe loop and MCP integration
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Deepseek-R1-70B
  role: Core LLM engine (DeepSeek-R1-Distill-Llama-70B on port 8080) used for planning,
    ontology filtering, categorization, and observation
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned concept extractor (port 8081) used by get_concept with few-shot
    prompting
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: BioPortal Annotator API
  role: Maps lemmatized concepts to SNOMED-CT and RADLEX ontologies in the ontology_mapping
    step
- name: mcp-python-sdk
  role: Model Context Protocol SDK implementing the mcp_client.py / mcp_server.py
    agent-server split
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python
  role: Runtime environment (Python 3.8 or higher) for the agent and tools
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:c7c0fb884a19063ab90962f9699adfaf4c7f2ae72c9b5d0bd9f02d46458996a0
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
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3063
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# plan-act-observe-agent-orchestration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An agentic orchestration pattern (MedPAO) that runs a Plan-Act-Observe (PAO) loop over an MCP-integrated toolset to decompose free-text medical report structuring into transparent, verifiable steps grounded in the ABCDEF protocol.

## When to use

Use when a free-text narrative (e.g., a radiology report) must be transformed into protocol-compliant structured data and a transparent, tool-orchestrated agent is preferable to a monolithic LLM call prone to hallucination.

## When NOT to use

- The input is already structured or a checklist-organized report, so no free-text-to-protocol mapping is needed
- The task requires real-time clinical deployment or integration with medical images — both are explicitly unimplemented future work in the paper
- No internet access is available, since the ontology_mapping step depends on the BioPortal Annotator API

## Inputs

- Free-text medical/radiology report (e.g., MIMIC-CXR chest X-ray findings passed into the {findings} prompt placeholder)
- Target clinical protocol description (ABCDEF for CXR analysis)
- BioPortal API key (apis.)
- Optional local cache of previously categorized concepts (cached_vocab.)

## Outputs

- concepts_output. — extracted concepts and their source sentences (get_concept)
- new_concepts_output. — concepts not found in the local cache (check_cache)
- ontologyMapping.csv — concepts with mapped SNOMED-CT/RADLEX ontologies and hierarchy
- segregated. — PRIMARY/SECONDARY ontology classification per concept (filter_ontologies)
- categorized_concepts. — concepts assigned to ABCDEF categories (categorize_concepts)
- categorized_report. — final structured report written to savings/

## How to apply

Deploy two local TGI model servers — DeepSeek-R1-Distill-Llama-70B as the LLM engine on port 8080 and the fine-tuned Med Llama-8B concept extractor on port 8081 — and set the BioPortal API key in apis.. In the Plan step, the MCP client (mcp_client.py) prompts the LLM engine with the available tools and previously executed steps so it selects the next tool and its arguments; the Act step executes tools in order: get_concept (few-shot Med Llama-8B extraction of concepts with source sentences), ontology_mapping via the BioPortal Annotator API to SNOMED-CT and RADLEX, filter_ontologies (few-shot LLM classification into PRIMARY and SECONDARY), categorize_concepts (LLM assigns each concept an ABCDEF category given the full protocol description), and generate_report. Optionally run check_cache first against cached_vocab. to reuse categorizations of recurring concepts and skip ontology mapping/filtering. In the Observe step, the LLM evaluates whether the tool output adequately addresses the original user query, iterating back to Plan if not, else terminating and writing the final structured report to savings/categorized_report..

## Related tools

- **MedPAO** (The agentic framework implementing the Plan-Act-Observe loop and MCP integration) — https://github.com/MiRL-IITM/medpao-agent
- **Deepseek-R1-70B** (Core LLM engine (DeepSeek-R1-Distill-Llama-70B on port 8080) used for planning, ontology filtering, categorization, and observation) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned concept extractor (port 8081) used by get_concept with few-shot prompting) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **BioPortal Annotator API** (Maps lemmatized concepts to SNOMED-CT and RADLEX ontologies in the ontology_mapping step)
- **mcp-python-sdk** (Model Context Protocol SDK implementing the mcp_client.py / mcp_server.py agent-server split) — https://github.com/MiRL-IITM/medpao-agent
- **Python** (Runtime environment (Python 3.8 or higher) for the agent and tools) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- The final categorized_report. contains all six ABCDEF keys populated per the protocol
- Report reproduces the Figure 4 reference mapping (e.g., 'clear lungs' and 'no pneumothorax' under B, 'normal heart size' under C)
- Concept categorization achieves an F1-score around 0.96 on the critical sub-task
- Expert structure scores fall in the reported range (4.48-4.59, average ~4.52 out of 5)
- Per-tool intermediate outputs appear in savings/ (concepts_output., ontologyMapping.csv, segregated., categorized_concepts.) confirming each PAO Act step executed

## Limitations

- LLM-based planning and few-shot classification remain prone to hallucination, which is why verifiable specialized tools are used instead of monolithic generation
- Integration with medical images is not yet implemented (proposed as future work)
- Real-time clinical deployment is not achieved; it depends on model compression and hardware acceleration optimizations
- Requires two local LLM servers (ports 8080 and 8081) and internet access for BioPortal API calls, limiting portability

## Evidence

- [methods] Caching rationale: [evidence span withheld — anchor status: not_found]
