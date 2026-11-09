---
name: topic-coherence-diversity-evaluation
description: Use when a topic list (e.g. topics or deduplicated_topics emitted by
  MALTopic's agents) must be judged against LDA/BERTopic baselines on survey-response
  data, to confirm refinement improved coherence, diversity, and interpretability.
when_to_use_negative:
- The topic list has not yet passed through the deduplication agent, so duplicate
  or overlapping themes are still present and comparison would be unfair
- The corpus is not survey-response text (the reported comparative analysis was performed
  on a survey dataset), so the coherence/diversity gains over LDA and BERTopic may
  not transfer
- A numeric evaluation protocol with explicit thresholds is required — the source
  states no numeric deduplication threshold and validation is qualitative
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MALTopic
  role: Multi-agent LLM framework whose refined topic lists are the object of the
    coherence/diversity/interpretability evaluation
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model against which MALTopic's coherence, diversity, and interpretability
    are compared
- name: openai-python
  role: Python client for the LLM backing MALTopic's agents (enrichment, topic modeling,
    deduplication) whose outputs are evaluated
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:80205098dd65016073f76b65215dd58d58a7139b60eb920c58a43d7223a59316
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# topic-coherence-diversity-evaluation

## Summary

Evaluates a topic-modeling pipeline's output by comparing topic coherence, diversity, and interpretability against LDA and BERTopic baselines on a survey dataset, as done for the MALTopic multi-agent LLM framework. It verifies that upstream refinement steps (notably the deduplication agent) actually yield distinct, human-readable topics.

## When to use

Use when a topic list (e.g. topics or deduplicated_topics emitted by MALTopic's agents) must be judged against LDA/BERTopic baselines on survey-response data, to confirm refinement improved coherence, diversity, and interpretability.

## When NOT to use

- The topic list has not yet passed through the deduplication agent, so duplicate or overlapping themes are still present and comparison would be unfair
- The corpus is not survey-response text (the reported comparative analysis was performed on a survey dataset), so the coherence/diversity gains over LDA and BERTopic may not transfer
- A numeric evaluation protocol with explicit thresholds is required — the source states no numeric deduplication threshold and validation is qualitative

## Inputs

- Topic list produced by the Topic Modeling Agent (Agent 2) of MALTopic
- Deduplicated topic list written to deduplicated_topics by the Topic Deduplication Agent (Agent 3)
- Survey-response dataset with structured metadata used by the enrichment agent
- Baseline topic outputs from LDA and BERTopic on the same survey dataset

## Outputs

- Comparative scores of topic coherence, diversity, and interpretability vs. LDA and BERTopic
- Qualitative judgment of whether the refined topic list is fully deduplicated and human-readable

## How to apply

Run the candidate pipeline end-to-end first: the enrichment agent leverages structured data to enhance textual responses, the topic modeling agent extracts latent themes, and the deduplication agent refines the results — invoking deduplication whenever the number of survey responses exceeds the Topic Modeling Agent's single-pass capacity, merging duplicate/overlapping themes into distinct representative topics. Then apply the comparative analysis protocol from the MALTopic paper: score topic coherence, topic diversity, and interpretability on the survey dataset and compare the scores against LDA and BERTopic run on the same data. Because the source reports no numeric deduplication threshold or metric cutoff, treat the validation as qualitative: acceptance requires that every entry in the refined list is distinct (no duplicate or semantically overlapping topics remain) and that the refined topics retain the human-readable, contextually relevant labels MALTopic is designed to produce.

## Related tools

- **MALTopic** (Multi-agent LLM framework whose refined topic lists are the object of the coherence/diversity/interpretability evaluation) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model against which MALTopic's coherence, diversity, and interpretability are compared)
- **openai-python** (Python client for the LLM backing MALTopic's agents (enrichment, topic modeling, deduplication) whose outputs are evaluated) — https://github.com/openai/openai-python

## Evaluation signals

- Every entry in the refined/deduplicated topic list is distinct — no duplicate or semantically overlapping topics remain
- MALTopic's coherence, diversity, and interpretability scores exceed those of LDA and BERTopic on the same survey dataset
- Refined topics retain human-readable labels with enhanced contextual relevance (enriched by structured data)
- The deduplicated_topics output exists and corresponds 1:1 to merged clusters from Agent 2's topic list
- Comparison is run on identical survey-response inputs across MALTopic, LDA, and BERTopic

## Limitations

- No numeric deduplication threshold is stated in the source, so acceptance of the refined topic list is qualitative rather than metric-driven
- Comparative evidence is limited to a single survey dataset; the reported improvements over LDA and BERTopic may not generalize to other corpora
- Deduplication is only triggered when the number of survey responses exceeds the Topic Modeling Agent's capacity for single-pass processing; for small inputs the dedup step may be unnecessary

## Evidence
