---
name: topic-deduplication-refinement
description: Use when a survey-dataset topic list from an upstream topic modeling
  agent contains duplicate or semantically overlapping themes — e.g. when response
  volume exceeded Agent 2's single-pass capacity — and a refined, distinct topic list
  is needed.
when_to_use_negative:
- The topic list already contains distinct entries with no duplicate or overlapping
  themes
- Agent 2 processed all survey responses in a single pass and deduplication is not
  required
- The goal is raw latent-theme extraction rather than refinement of an existing topic
  list
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: MALTopic
  role: The multi-agent LLM framework whose Agent 3 (deduplication agent) performs
    this refinement step on Agent 2's topic list
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic modeling method that MALTopic's deduplicated topic output is
    compared against for coherence, diversity, and interpretability
- name: openai-python
  role: Python client used to access OpenAI LLM APIs that power the LLM-based multi-agent
    pipeline including the deduplication agent
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:cd7da5c30a2a9a50abb4ec4e4798f28e75a37d58b0a16087d9b685c9f6ad292b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# topic-deduplication-refinement

## Summary

Refines the topic list produced by the Topic Modeling Agent (Agent 2) of the MALTopic multi-agent LLM framework by having a LLM-based deduplication agent merge duplicate and semantically overlapping topics into distinct, human-readable labels.

## When to use

Use when a survey-dataset topic list from an upstream topic modeling agent contains duplicate or semantically overlapping themes — e.g. when response volume exceeded Agent 2's single-pass capacity — and a refined, distinct topic list is needed.

## When NOT to use

- The topic list already contains distinct entries with no duplicate or overlapping themes
- Agent 2 processed all survey responses in a single pass and deduplication is not required
- The goal is raw latent-theme extraction rather than refinement of an existing topic list

## Inputs

- Topic list (labels/keywords) emitted by the Topic Modeling Agent (Agent 2) of the MALTopic framework
- Enriched survey responses processed earlier in the MALTopic pipeline (implicit context for the topics)

## Outputs

- Refined topic list written to deduplicated_topics, containing only distinct topics with human-readable labels

## How to apply

Load the topic list emitted by the upstream Topic Modeling Agent of the MALTopic framework, which extracts latent themes from enriched survey responses. Run the deduplication agent over this list, comparing topic labels and keywords pairwise to identify duplicate and semantically overlapping themes. Merge each cluster of duplicate/overlapping topics into a single representative topic, preserving the human-readable, contextually relevant labels MALTopic is designed to produce. Write the refined list to deduplicated_topics. The article does not state a numeric deduplication threshold, so accept the output qualitatively: every entry must be distinct and the refined list should support the improved coherence, diversity, and interpretability MALTopic reports over LDA and BERTopic.

## Related tools

- **MALTopic** (The multi-agent LLM framework whose Agent 3 (deduplication agent) performs this refinement step on Agent 2's topic list) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic modeling method that MALTopic's deduplicated topic output is compared against for coherence, diversity, and interpretability)
- **openai-python** (Python client used to access OpenAI LLM APIs that power the LLM-based multi-agent pipeline including the deduplication agent) — https://github.com/openai/openai-python

## Evaluation signals

- Every entry in the refined deduplicated_topics list is distinct — no duplicate or overlapping topics remain
- Refined topics retain human-readable labels with enhanced contextual relevance, consistent with MALTopic's design
- Refined topic list supports the reported improvement in topic coherence, diversity, and interpretability relative to LDA and BERTopic on the survey dataset
- Refined list is produced only when the number of survey responses exceeded Agent 2's single-pass processing capacity

## Limitations

- No numeric deduplication threshold is stated in the source, so acceptance of the deduplication result is qualitative
- Deduplication is required only when survey response volume exceeds Agent 2's capacity for single-pass processing; otherwise it may be unnecessary
- Success is measured by comparison to LDA and BERTopic baselines on a survey dataset; behavior on other data types is not characterized

## Evidence
