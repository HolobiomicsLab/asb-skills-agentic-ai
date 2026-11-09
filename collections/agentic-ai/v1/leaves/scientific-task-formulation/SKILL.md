---
name: scientific-task-formulation
description: Use when a research question targets architecture or mechanisms (e.g.
  AgriIR's duplicate-tracking, content-hashing, quality-scoring) that may not be documented
  in the paper text, so scope must be validated against repos and datasets before
  execution.
when_to_use_negative:
- The target mechanism is already fully documented in the article's section text (e.g.
  the 6-stage AgriIR RAG pipeline) — extract entities directly instead of reformulating
  scope
- The question asks for quantitative benchmark results (e.g. composite scores like
  0.820±0.208 for AgriIR_Gemma-3-27B) rather than architecture reconstruction
- No repository, code, or dataset artifact is available to widen scope when the paper
  text is insufficient
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AgriIR (framework)
  role: The system whose architecture is being formulated as an executable task; its
    modular declarative stages define the component vocabulary used in task scoping
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: XAheli/AgriIR repository
  role: Repository evidence source for the database pipeline (database_pipeline.png)
    used to widen task scope when section text omits the agentic database-creation
    components
  repo: https://github.com/XAheli/AgriIR
- name: ShuvBan/AgriIR_dataset
  role: Artifact (15,247 agricultural knowledge entries) that anchors the database-creation
    output of the formulated task
  repo: https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:217180535a3a36debc536d59f441b0b136b66e69fb762c4dce8d9683f2a8b041
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR (framework)
  - ShuvBan/AgriIR_dataset
  - XAheli/AgriIR repository
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scientific_task_formulation

## Summary

Formulates a natural-language research question into a scoped, executable scientific task by checking which components the available evidence (section text, READMEs, code, datasets) actually covers, and by explicitly flagging parts that are not executable as scoped. Here it is applied to reconstructing AgriIR's agentic database-creation architecture (duplicate tracking, JSONL persistence, quality-driven learning) from article and repository evidence.

## When to use

Use when a research question targets architecture or mechanisms (e.g. AgriIR's duplicate-tracking, content-hashing, quality-scoring) that may not be documented in the paper text, so scope must be validated against repos and datasets before execution.

## When NOT to use

- The target mechanism is already fully documented in the article's section text (e.g. the 6-stage AgriIR RAG pipeline) — extract entities directly instead of reformulating scope
- The question asks for quantitative benchmark results (e.g. composite scores like 0.820±0.208 for AgriIR_Gemma-3-27B) rather than architecture reconstruction
- No repository, code, or dataset artifact is available to widen scope when the paper text is insufficient

## Inputs

- Research question string (e.g. how AgriIR's agentic database creation handles duplicate tracking and quality-driven learning)
- Source cards with finding and workflow fields per task_id
- Article section text with extracted entities (tools, workflow_steps, filter_steps, findings)
- Repository READMEs and repo links (Shuvam-Banerji-Seal/AgriIR, XAheli/AgriIR, XAheli/AgriIR_Query_Gen)
- Linked artifacts (HuggingFace dataset ShuvBan/AgriIR_dataset)

## Outputs

- Scoped task definition stating which components are executable as scoped and which are not
- Workflow narrative enumerating present components (6-stage RAG pipeline, dataset distribution, installation/config) and absent components (duplicate-tracking, content-hashing, PDF-fallback, quality-scoring)
- Tool selection record (e.g. manual expert review)
- Sharable finding sentence with traced evidence spans

## How to apply

Start by decomposing the research question into concrete mechanisms to verify (for AgriIR: persistent duplicate tracking, real-time JSONL logging, quality-driven learning of the agentic database pipeline). Then map each mechanism to evidence sources: the article's 6-stage pipeline sections (Query Refinement T=0.1, Sub-Query T=0.5, Parallel Retrieval, Agent Enhancement, Synthesis T=0.2, Citation Insert at similarity >= 0.75), the repository README/code (e.g. github.com/XAheli/AgriIR/database_pipeline.png), and artifacts like the ShuvBan/AgriIR_dataset with 15,247 entries. At the decision point, check whether the section text supports the mechanism; if not, the task is marked 'not executable as scoped' and the evidence-gathering scope is widened to repository code and READMEs rather than fabricating findings from adjacent sections. Output a scoped finding plus an explicit workflow stating what was found, what was missing, and which review method (manual expert review) was used.

## Related tools

- **AgriIR (framework)** (The system whose architecture is being formulated as an executable task; its modular declarative stages define the component vocabulary used in task scoping) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **XAheli/AgriIR repository** (Repository evidence source for the database pipeline (database_pipeline.png) used to widen task scope when section text omits the agentic database-creation components) — https://github.com/XAheli/AgriIR
- **ShuvBan/AgriIR_dataset** (Artifact (15,247 agricultural knowledge entries) that anchors the database-creation output of the formulated task) — https://huggingface.co/datasets/ShuvBan/AgriIR_dataset

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- Every mechanism named in the research question is either located in evidence or explicitly listed as absent (e.g. 'none of the duplicate-tracking, content-hashing, PDF-fallback, or quality-scoring components appear in it')
- Workflow distinguishes components grounded in section text from those requiring repository/README evidence
- Claimed artifacts are verifiable at the cited links (repo clone, database_pipeline.png, HuggingFace dataset with 15,247 entries)
- The workflow declares the verification method used (manual expert review) rather than implying automated extraction
- No findings are inferred from adjacent topics (e.g. RAG pipeline stages) to fill gaps about database-creation internals

## Limitations

- The article section text covers only the 6-stage AgriIR RAG pipeline, dataset distribution, and installation/configuration — duplicate-tracking, content-hashing, PDF-fallback, and quality-scoring components do not appear in it, so reconstruction depends on repository code and READMEs
- No discussion section text is present in the source document, so limitations, missing artifacts, and reproducibility gaps cannot be extracted from the paper itself
- Repository documentation is truncated, leaving some pipeline details (e.g. full database-creation steps) unverifiable from READMEs alone

## Evidence

- [other] Section text describes only the 6-stage pipeline; agentic database-creation components are absent: [evidence span withheld — anchor status: not_found]
