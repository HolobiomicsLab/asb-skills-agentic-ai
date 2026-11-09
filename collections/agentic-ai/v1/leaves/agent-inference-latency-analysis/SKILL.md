---
name: agent-inference-latency-analysis
description: Use when you need to decompose an agentic LLM pipeline's end-to-end runtime
  into per-tool latencies, or validate a reported caching speedup such as MedPAO's
  280 s vs. 66 s comparison.
when_to_use_negative:
- When the goal is accuracy evaluation (e.g., the 0.96 F1-score on concept categorization)
  rather than runtime measurement
- When the deployment hardware or serving stack differs from the Gaudi2 TGI setup
  (6-shard 70B engine, TGI 3.2.3-gaudi image), since latencies are hardware-specific
  and the paper notes real-time clinical deployment still depends on model compression
  and hardware acceleration
- When the input is already a structured report or feature table and no free-text-to-structured
  agent inference is being timed
tools:
- name: MedPAO
  role: The Protocol-Driven Agent whose Plan-Act-Observe loop orchestrates the specialized
    tools whose per-tool latencies are tabulated
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: DeepSeek-R1-Distill-Llama-70B
  role: Core LLM engine on port 8080 whose inference drives planning and report_generation
    latencies; sharded across 6 Gaudi2 devices
  repo: https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- name: Med Llama-8B
  role: Fine-tuned concept extractor on port 8081 whose inference constitutes the
    get_concept tool's latency
  repo: https://huggingface.co/shrishSVaidya/medllama_finetuned
- name: check_cache
  role: Caching tool that, when enabled, skips ontology_mapping, ontology_filtering,
    and concept_categorization for recurring concepts, producing the 66 s variant
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python 3.8
  role: Runtime used to run mcp_client.py and to encode the timing values into a structured
    CSV table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:544dfce3a07990c3b265478d91eab436f26053e81f142c617937d177b2af4b9e
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
  - DeepSeek-R1-Distill-Llama-70B
  - Med Llama-8B
  - MedPAO
  - Python
  - Python 3.8
  - check_cache
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# agent-inference-latency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for reconstructing and validating the per-tool inference-timing breakdown of MedPAO's Plan-Act-Observe pipeline on the freestyle report-to-structured-report task, reproducing the published 280 s end-to-end runtime and the 66 s runtime when the check_cache tool is enabled.

## When to use

Use when you need to decompose an agentic LLM pipeline's end-to-end runtime into per-tool latencies, or validate a reported caching speedup such as MedPAO's 280 s vs. 66 s comparison.

## When NOT to use

- When the goal is accuracy evaluation (e.g., the 0.96 F1-score on concept categorization) rather than runtime measurement
- When the deployment hardware or serving stack differs from the Gaudi2 TGI setup (6-shard 70B engine, TGI 3.2.3-gaudi image), since latencies are hardware-specific and the paper notes real-time clinical deployment still depends on model compression and hardware acceleration
- When the input is already a structured report or feature table and no free-text-to-structured agent inference is being timed

## Inputs

- Freestyle chest X-ray report text from the Eye Gaze MIMIC-CXR dataset (40 preliminary reports), passed via the {findings} placeholder of the mcp_client.py prompt
- Per-tool inference-time table entries for the agent run (planning, get_concept, ontology_mapping, ontology_filtering, concept_categorization, report_generation, overall)
- cached_vocab. local cache of previously processed concepts and their ABCDEF protocol categorizations

## Outputs

- Structured CSV table with per-tool latency columns (planning, get_concept, ontology_mapping, ontology_filtering, concept_categorization, report_generation) and an overall agent-time column for both the no-cache (280 s) and check_cache-enabled (66 s) runs
- Validated reproduction of the published Table 5 timing breakdown

## How to apply

Serve the DeepSeek-R1-Distill-Llama-70B engine on port 8080 and the fine-tuned Med Llama-8B concept extractor on port 8081 via HuggingFace TGI (ghcr.io/huggingface/text-generation-inference:3.2.3-gaudi, --sharded true --num-shard 6), then run mcp_client.py on chest X-ray reports from the Eye Gaze MIMIC-CXR set. Without caching, tabulate per-tool times: planning 46 s, get_concept 1 s, ontology_mapping 56 s, ontology_filtering 132 s, concept_categorization 25 s, report_generation 20 s, summing to 280 s. With the check_cache tool specified in the prompt, ontology_mapping/filtering/categorization are not executed, and the timing row becomes planning 45 s, get_concept 1 s, report_generation 20 s, summing to 66 s. Encode both timing rows with column labels for each tool plus the overall agent time into a structured CSV table using Python 3.8, and verify every cell and both totals match the published Table 5 values exactly.

## Related tools

- **MedPAO** (The Protocol-Driven Agent whose Plan-Act-Observe loop orchestrates the specialized tools whose per-tool latencies are tabulated) — https://github.com/MiRL-IITM/medpao-agent
- **DeepSeek-R1-Distill-Llama-70B** (Core LLM engine on port 8080 whose inference drives planning and report_generation latencies; sharded across 6 Gaudi2 devices) — https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B
- **Med Llama-8B** (Fine-tuned concept extractor on port 8081 whose inference constitutes the get_concept tool's latency) — https://huggingface.co/shrishSVaidya/medllama_finetuned
- **check_cache** (Caching tool that, when enabled, skips ontology_mapping, ontology_filtering, and concept_categorization for recurring concepts, producing the 66 s variant) — https://github.com/MiRL-IITM/medpao-agent
- **Python 3.8** (Runtime used to run mcp_client.py and to encode the timing values into a structured CSV table)

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, without using check_cache tool: {findings}"
```

## Evaluation signals

- Sum of per-tool latencies in the no-cache run equals the published overall agent time exactly (46 + 1 + 56 + 132 + 25 + 20 = 280 s)
- Sum of per-tool latencies in the check_cache-enabled run equals 66 s (45 + 1 + 20), with ontology_mapping, ontology_filtering, and concept_categorization rows marked as not executed
- Every encoded CSV cell matches the published Table 5 value exactly on cell-by-cell validation
- The cached run produces new_concepts_output. containing only concepts absent from cached_vocab., confirming the ontology tools were genuinely bypassed
- Each tool's expected output file appears in the savings/ directory (concepts_output., ontologyMapping.csv, segregated., categorized_concepts., categorized_report.), confirming all timed tools actually executed

## Limitations

- Timings are hardware- and serving-specific (Gaudi2 accelerators with TGI 3.2.3-gaudi, --num-shard 6 for the 70B engine); they will not transfer to other hardware
- The caching speedup only applies to recurring medical concepts already present in cached_vocab.; novel concepts still require the full ontology pipeline
- Real-time clinical deployment is not yet achieved; the paper notes it depends on model compression and hardware acceleration optimizations
- Timing measurement relies on BioPortal Annotator API calls for ontology_mapping, so network latency can affect the reported 56 s figure

## Evidence

- [methods] timing breakdown workflow: "Our agent employs a Plan-Act-Observe (PAO) loop [30] to orchestrate specialized, verifiable tools, systematically mapping narrative findings back to a clinical protocol"
- [intro] toolset composition: "We introduce a modular and verifiable toolset, including a fine-tuned medical concept extractor and reasoning-based modules for ontology filtering and protocol categorization."
- [methods] cache mechanism: [evidence span withheld — anchor status: not_found]
- [readme] cache-enabled invocation: "python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}""
- [readme] engine serving configuration: "ghcr.io/huggingface/text-generation-inference:3.2.3-gaudi    --model-id $model    --sharded true --num-shard 6"
