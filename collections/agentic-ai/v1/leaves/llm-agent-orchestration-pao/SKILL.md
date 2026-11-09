---
name: llm-agent-orchestration-pao
description: Use when an LLM pipeline must be decomposed into transparent, verifiable
  tool steps whose sequence should be dynamically decided by the agent rather than
  hard-coded, e.g. structuring freestyle medical reports into protocol-compliant JSON.
when_to_use_negative:
- The tool sequence must be fixed and fully deterministic with no agent discretion
  — a hard-coded script is simpler and more reproducible.
- Inputs are medical images or modalities other than free-text clinical narratives;
  image integration is proposed as future work and not implemented.
- Real-time clinical deployment is required; the framework depends on model compression
  and hardware acceleration optimizations not yet achieved.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3386
- http://edamontology.org/topic_3474
tools:
- name: MedPAO
  role: The protocol-driven agent whose PAO loop orchestrates the specialized verifiable
    tools
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: mcp-python-sdk
  role: SDK used to implement the MCP client/server pair that exposes tools to the
    agent
- name: Deepseek-R1-70B
  role: Core LLM engine performing planning and reasoning-based tool orchestration
    on port 8080
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned model served on port 8080's peer port 8081, invoked by the get_concept
    tool during the orchestrated loop
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: Python 3.8
  role: Language implementing mcp_client.py, mcp_server.py and each tool module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:0d28d7ccfde43ab2d1618e8bbc5d3e0318b0bb2ca69c39c3b77044487e4efb44
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
  - Python 3.8
  - mcp-python-sdk
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3386
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# llm-agent-orchestration-pao

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Orchestrate LLM-driven scientific task pipelines with a Plan-Act-Observe (PAO) loop over modular, verifiable tools via the Model Context Protocol (MCP), as in MedPAO's client/server agent that decomposes report structuring into tool calls (get_concept, ontology_mapping, filter_ontology, categorize_concepts, generate_report, check_cache).

## When to use

Use when an LLM pipeline must be decomposed into transparent, verifiable tool steps whose sequence should be dynamically decided by the agent rather than hard-coded, e.g. structuring freestyle medical reports into protocol-compliant JSON.

## When NOT to use

- The tool sequence must be fixed and fully deterministic with no agent discretion — a hard-coded script is simpler and more reproducible.
- Inputs are medical images or modalities other than free-text clinical narratives; image integration is proposed as future work and not implemented.
- Real-time clinical deployment is required; the framework depends on model compression and hardware acceleration optimizations not yet achieved.

## Inputs

- Free-text medical/radiology report (e.g. MIMIC-CXR findings) passed via the --prompt {findings} placeholder
- Task prompt specifying the target clinical protocol and whether to use check_cache
- BioPortal API key in apis.
- Local LLM servers on ports 8080 (reasoning engine) and 8081 (fine-tuned concept extractor)

## Outputs

- savings/concepts_output. (concepts + source sentences from get_concept)
- savings/ontologyMapping.csv (concept-to-SNOMED-CT/RADLEX mappings from ontology_mapping)
- savings/segregated. (PRIMARY/SECONDARY ontology classification from filter_ontology)
- savings/categorized_concepts. (protocol-section categorization from categorize_concepts)
- savings/categorized_report. (final structured report from generate_report)
- updated cached_vocab. of recurring concepts and their categories

## How to apply

Implement an MCP client (mcp_client.py) paired with an MCP server (mcp_server.py) that exposes specialized tools, and let a Plan-Act-Observe loop over a reasoning LLM engine (DeepSeek-R1-Distill-Llama-70B served via Huggingface TGI on port 8080) decide the tool sequence rather than hard-coding it. Each tool is a standalone Python module in tools/ (get_concept, ontology_mapping, filter_ontology, categorize_concepts, generate_report, check_cache) whose intermediate output is persisted to savings/ so every step is inspectable. The agent prompt states the protocol-driven task (e.g. 'structure the given medical report according to ABCDEF protocol') and whether to use the check_cache tool for recurring concept-category lookups stored in cached_vocab.. Rationale: the PAO loop grounds LLM operation in specialized verifiable tools, avoiding opaque monolithic LLM calls prone to hallucination, and the modular toolset (reasoning-based ontology filtering and protocol categorization) is what allows the pipeline to be re-instantiated with a different clinical protocol beyond ABCDEF.

## Related tools

- **MedPAO** (The protocol-driven agent whose PAO loop orchestrates the specialized verifiable tools) — https://github.com/MiRL-IITM/medpao-agent
- **mcp-python-sdk** (SDK used to implement the MCP client/server pair that exposes tools to the agent)
- **Deepseek-R1-70B** (Core LLM engine performing planning and reasoning-based tool orchestration on port 8080) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned model served on port 8080's peer port 8081, invoked by the get_concept tool during the orchestrated loop) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **Python 3.8** (Language implementing mcp_client.py, mcp_server.py and each tool module)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- categorized_report. exists in savings/, is valid JSON, and contains a distinct section for every category of the target protocol (e.g. each ABCDEF category).
- Intermediate per-tool outputs (concepts_output., ontologyMapping.csv, segregated., categorized_concepts.) are all present, showing the PAO loop completed each Act-Observe step.
- Concept categorization sub-task reaches the paper's benchmark F1-score of 0.96.
- Final structured reports meet the expert-radiologist benchmark mean accuracy score of 4.52 out of 5.
- check_cache path: new_concepts_output. contains only concepts absent from cached_vocab., and cached_vocab. grows across samples.

## Limitations

- The demonstrated instantiation targets chest X-ray reports via the ABCDEF/P.A.B.C.D.E.F protocol on MIMIC-CXR; generalization to other protocols relies on the modular toolset but was not fully demonstrated in the paper.
- Integration with medical images is not implemented (future work).
- Real-time clinical deployment is not yet achieved and may require model compression and hardware acceleration optimizations.
- Requires two local LLM inference servers via the Huggingface TGI framework (e.g. on Gaudi2 accelerators) and internet access for BioPortal API calls, so it is not trivially portable to resource-constrained environments.

## Evidence
