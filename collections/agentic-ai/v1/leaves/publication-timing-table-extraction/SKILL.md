---
name: publication-timing-table-extraction
description: Use when a paper reports a per-component runtime table (tool-by-tool
  latencies plus an overall total) and the task is to reproduce, tabulate, or verify
  the reported end-to-end inference timing and caching speedup.
when_to_use_negative:
- The paper reports only aggregate runtime with no per-tool latency breakdown to transcribe
- The goal is to measure new timings by running the agent rather than extracting and
  validating already-published Table 5 values
- The task targets a different MedPAO protocol variant or dataset for which no timing
  table was published
tools:
- name: MedPAO
  role: The agent whose Plan-Act-Observe pipeline's per-tool latencies are tabulated
    (planning, get_concept, ontology_mapping, filter_ontology, categorize_concepts,
    generate_report, check_cache)
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python 3.8
  role: Runtime used (with pandas) to encode the extracted timing values into a structured
    CSV table and validate the overall totals
- name: check_cache
  role: Caching tool whose activation produces the second timing configuration (66
    s overall) by skipping ontology_mapping, filtering, and categorization for cached
    concepts
  repo: https://github.com/MiRL-IITM/medpao-agent
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:c64527db765164eeb3a628cc75316bd9a2381e2a354a61c5b2d7c32b6e37b8c2
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
  - MedPAO
  - Python
  - Python 3.8
  - check_cache
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# publication-timing-table-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and re-encodes per-tool inference-time breakdowns published in a paper's results tables (here, MedPAO's Table 5: 280 s without caching vs. 66 s with check_cache) into a structured CSV so end-to-end agent runtimes can be reconstructed and validated cell-by-cell.

## When to use

Use when a paper reports a per-component runtime table (tool-by-tool latencies plus an overall total) and the task is to reproduce, tabulate, or verify the reported end-to-end inference timing and caching speedup.

## When NOT to use

- The paper reports only aggregate runtime with no per-tool latency breakdown to transcribe
- The goal is to measure new timings by running the agent rather than extracting and validating already-published Table 5 values
- The task targets a different MedPAO protocol variant or dataset for which no timing table was published

## Inputs

- Published inference-time table (Table 5) values for the freestyle report-to-structured-report task, without caching (280 s total)
- Published inference-time values with check_cache enabled (66 s total)
- List of MedPAO pipeline tools: planning, get_concept, ontology_mapping, ontology_filtering, concept_categorization, report_generation, overall

## Outputs

- CSV table (Python 3.8 / pandas) with one row per configuration (cache off / cache on) and columns for each tool's latency in seconds plus the overall agent time
- Verified totals: 280 s (no cache) and 66 s (check_cache enabled), matching the published table

## How to apply

Read the published inference-time table for the target task (MedPAO's freestyle report-to-structured-report on the Eye Gaze MIMIC-CXR chest X-ray reports) and transcribe each specialized tool's latency: planning 46 s, get_concept 1 s, ontology_mapping 56 s, ontology_filtering 132 s, concept_categorization 25 s, report_generation 20 s, overall 280 s. Then transcribe the caching-enabled variant (planning 45 s, get_concept 1 s, ontology_mapping/filtering/categorization skipped via check_cache, report_generation 20 s, overall 66 s). Encode both rows, with column labels naming each tool and the overall agent time, into a CSV using Python 3.8 (pandas). Sum the per-tool columns to confirm they equal the reported totals (46+1+56+132+25+20=280; 45+1+20=66) before treating the table as correctly reproduced.

## Related tools

- **MedPAO** (The agent whose Plan-Act-Observe pipeline's per-tool latencies are tabulated (planning, get_concept, ontology_mapping, filter_ontology, categorize_concepts, generate_report, check_cache)) — https://github.com/MiRL-IITM/medpao-agent
- **Python 3.8** (Runtime used (with pandas) to encode the extracted timing values into a structured CSV table and validate the overall totals)
- **check_cache** (Caching tool whose activation produces the second timing configuration (66 s overall) by skipping ontology_mapping, filtering, and categorization for cached concepts) — https://github.com/MiRL-IITM/medpao-agent

## Examples

```
python mcp_client.py --prompt "the task is to structure the given medical report according to ABCDEF protocol, utilizing the check_cache tool: {findings}"
```

## Evaluation signals

- Row sum invariant: planning 46 + get_concept 1 + ontology_mapping 56 + ontology_filtering 132 + concept_categorization 25 + report_generation 20 equals the published overall 280 s
- Caching row invariant: planning 45 + get_concept 1 + report_generation 20 equals the published overall 66 s, with ontology_mapping/filtering/categorization marked as not executed
- Every encoded CSV cell matches the published Table 5 value exactly (no rounded or re-estimated numbers)
- Speedup check: cached configuration is ~4.2x faster than the uncached 280 s run, consistent with the reported 280 s vs. 66 s comparison
- CSV schema check: columns labeled for each tool plus overall agent time, one row per caching configuration

## Limitations

- Timings are specific to the reported hardware setup (Gaudi2 accelerator running DeepSeek-R1-Distill-Llama-70B via TGI on port 8080 and the fine-tuned Med Llama-8B on port 8081) and will not transfer to other infrastructure
- The caching speedup assumes recurring concepts are already present in cached_vocab.; on a cold cache the ontology tools execute and the 66 s figure does not apply
- Reported values are single published numbers, not distributions; no variance or repetition count is available from the table

## Evidence
