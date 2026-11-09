---
name: survey-taxonomy-structure-extraction
description: Use when a survey or paper list expresses its contributions as a staged
  lifecycle/taxonomy (e.g. §2.2–§6.3 headings plus Figure 3 subcategories) that must
  be reconstructed as a structured framework record.
when_to_use_negative:
- The source document is a methodological paper describing a single technique rather
  than a survey organized into a stage taxonomy with numbered subsections
- The target structure is a numeric benchmark or evaluation result rather than a qualitative
  heading/section hierarchy — this skill has no numeric acceptance criterion
- Only the abstract is available; the full section headings (§2.2–§6.3) and Figure
  3 subcategories are needed to reconstruct the loop
tools:
- name: Python
  role: Parsing section headings with re and serializing the nested taxonomy record
    via the  module
- name: Zesearch/self-improvement-llm
  role: Source repository whose section-annotated paper list (§2 Data Acquisition
    through §6.3 Interactive Environment Evaluation) is parsed to build the taxonomy
  repo: https://github.com/Zesearch/self-improvement-llm
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:d9d936b4f0250549e3b01bd649144cfaef412c6b43a26d3f71ac9984c6c2801d
  source_papers:
  - doi: 10.48550/arxiv.2603.25681
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Zesearch/self-improvement-llm
  tools:
  - Python
  - Zesearch/self-improvement-llm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# survey-taxonomy-structure-extraction

## Summary

Recovers a survey's hierarchical taxonomy (section headings, subsections, and representative methods) into a machine-readable nested JSON record encoding the paper's unified closed-loop self-improvement lifecycle: Data Acquisition, Data Selection, Model Optimization, Inference Refinement, and Autonomous Evaluation.

## When to use

Use when a survey or paper list expresses its contributions as a staged lifecycle/taxonomy (e.g. §2.2–§6.3 headings plus Figure 3 subcategories) that must be reconstructed as a structured framework record.

## When NOT to use

- The source document is a methodological paper describing a single technique rather than a survey organized into a stage taxonomy with numbered subsections
- The target structure is a numeric benchmark or evaluation result rather than a qualitative heading/section hierarchy — this skill has no numeric acceptance criterion
- Only the abstract is available; the full section headings (§2.2–§6.3) and Figure 3 subcategories are needed to reconstruct the loop

## Inputs

- Survey paper text or repository paper list annotated with section numbers (§2.2–§6.3), e.g. the Zesearch/self-improvement-llm README paper list
- Figure 3 subcategory hierarchy from the survey
- Representative-method list per subcategory from the paper

## Outputs

- Nested taxonomy record serialized to self_improvement_framework_taxonomy (JSON) with five top-level stages, subcategory headings, and representative method names per subcategory

## How to apply

Parse the survey's section headings and stage descriptions (here §2.2 through §6.3) from the paper or repository paper-list using Python with the re module to recover the lifecycle components and their subsection hierarchy. Build the top-level closed-loop structure with the five stages — Data Acquisition (§2: Static Curation, Environment Interaction, Synthetic Generation), Data Selection (§3: Metric-Guided Scoring, Adaptive Selection), Model Optimization (§4: Direct, Self-Generated, Self-Evolving Optimization), Inference Refinement (§5: Decoding Strategies, Reasoning-Based, Agentic System-Based, Test-Time Training), and Autonomous Evaluation (§6: Dynamic Benchmarking, Interactive Environment Evaluation) — marking the autonomous evaluation layer as the monitoring/guidance component of the loop rather than a linear stage. Nest subcategories from Figure 3 (e.g. §2.4.1 Prompt-Based, §2.4.2 Transformation-Based, §2.4.3 Interaction-Based under Synthetic Generation; §5.2.1 Sampling-Based, §5.2.2 Tree Search under Decoding Strategies) and attach representative method names (e.g. WebGPT, Toolformer, MATES, STaR, SPIN, Self-Consistency, ReAct, TTRL) extracted from the paper list. Serialize the nested record with Python's  module to self_improvement_framework_taxonomy. Validate that the output loads, contains exactly the five stages, and that every subcategory heading from §2.2 to §6.3 is present with at least one representative method.

## Related tools

- **Python** (Parsing section headings with re and serializing the nested taxonomy record via the  module)
- **Zesearch/self-improvement-llm** (Source repository whose section-annotated paper list (§2 Data Acquisition through §6.3 Interactive Environment Evaluation) is parsed to build the taxonomy) — https://github.com/Zesearch/self-improvement-llm

## Examples

```
import ; .dump(taxonomy, open('self_improvement_framework_taxonomy.','w'), indent=2)
```

## Evaluation signals

- Output JSON loads without error via .load
- Record contains exactly the five top-level stages: Data Acquisition, Data Selection, Model Optimization, Inference Refinement, Autonomous Evaluation
- Every subcategory heading from §2.2 to §6.3 (e.g. Static Curation §2.2, Metric-Guided Scoring §3.2, Test-Time Training §5.5, Dynamic Benchmarking §6.2) appears in the nested record
- Every subcategory node carries at least one representative method name (e.g. WebGPT, STaR, SPIN, ReAct, TTRL)
- The autonomous evaluation layer is encoded as the monitoring/guidance component of the closed loop, not as a fifth linear stage

## Limitations

- Validation is a qualitative completeness check only — the paper reports no numeric criterion for the taxonomy itself
- The self-improvement framing assumes the lifecycle is closed-loop and five-component; surveys with different organizations (e.g. localized mechanisms rather than a system-level lifecycle) do not map onto this structure
- The section hierarchy must be reconstructed from headings and Figure 3; a truncated paper list or missing figure subcategories will leave gaps in the record

## Evidence

- [abstract] the autonomous evaluation layer continuously monitors progress and guides the improvement cycle across stages: "the autonomous evaluation layer continuously monitors progress and guides the improvement cycle across stages"
- [intro] we propose a lifecycle self-improvement system consisting of five interconnected components.: "we propose a lifecycle self-improvement system consisting of five interconnected components."
- [readme] We organize existing research into **five key components** of a self-improvement system:: "We organize existing research into **five key components** of a self-improvement system:"
