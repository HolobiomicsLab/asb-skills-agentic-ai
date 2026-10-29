---
name: topic-overlap-merge
description: Use when a topic-modeling run over survey responses yields a topic list
  containing duplicate or semantically overlapping themes — e.g., when the response
  volume exceeded Agent 2's single-pass capacity and the raw topic list must be refined
  before use.
when_to_use_negative:
- The topic list already contains only distinct, non-overlapping entries from a single-pass
  Agent 2 run — deduplication would add cost with no refinement
- You need a reproducible, threshold-based similarity merge (e.g., cosine similarity
  > 0.7); the article specifies no numeric deduplication threshold, so this step is
  qualitative
- The input is raw survey text that has not yet passed MALTopic's enrichment and topic
  modeling agents — this skill operates on the topic list, not the corpus
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: MALTopic
  role: Multi-agent LLM framework whose Topic Deduplication Agent (Agent 3) performs
    the overlap merge on the topic list produced by its Topic Modeling Agent
  repo: yash91sharma/MALTopic
- name: openai-python
  role: Python SDK providing the LLM chat completion calls through which the deduplication
    agent compares and merges topic labels
  repo: openai/openai-python
- name: BERTopic
  role: Baseline topic modeling method against which MALTopic's deduplicated, refined
    topic output is evaluated for coherence, diversity, and interpretability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:fab86894b9bfa0e2b80e7c89a3030aab9d0fef11744dc185623758a9f64d63c3
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
yes_rate: 1.0
n_claims: 7
---

# topic-overlap-merge

## Summary

Runs the MALTopic Topic Deduplication Agent (Agent 3), an LLM-based refinement step that merges duplicate and semantically overlapping topics from the Topic Modeling Agent's (Agent 2) topic list into a single refined, human-readable topic list.

## When to use

Use when a topic-modeling run over survey responses yields a topic list containing duplicate or semantically overlapping themes — e.g., when the response volume exceeded Agent 2's single-pass capacity and the raw topic list must be refined before use.

## When NOT to use

- The topic list already contains only distinct, non-overlapping entries from a single-pass Agent 2 run — deduplication would add cost with no refinement
- You need a reproducible, threshold-based similarity merge (e.g., cosine similarity > 0.7); the article specifies no numeric deduplication threshold, so this step is qualitative
- The input is raw survey text that has not yet passed MALTopic's enrichment and topic modeling agents — this skill operates on the topic list, not the corpus

## Inputs

- Topic list emitted by the MALTopic Topic Modeling Agent (Agent 2), containing topic labels and keywords derived from enriched survey responses
- Survey-response corpus processed by MALTopic's enrichment agent and topic modeling agent

## Outputs

- Deduplicated topic list written to deduplicated_topics, with each entry a distinct representative topic carrying a refined human-readable label

## How to apply

Load the topic list emitted by the upstream Topic Modeling Agent of the MALTopic framework, then run the LLM-based deduplication agent over it, comparing topic labels/keywords pairwise to identify duplicate and semantically overlapping themes. Merge each cluster of duplicate/overlapping topics into a single representative topic, preserving the human-readable, contextually relevant labels MALTopic is designed to produce (the agent 'refines the results' in the pipeline after enrichment and topic modeling). Write the merged output to deduplicated_topics. Note that the paper states no numeric deduplication threshold, so merging decisions are qualitative — judge overlap from label/keyword semantics rather than a fixed similarity cutoff. The deduplication step is specifically required when the number of survey responses exceeds Agent 2's capacity for single-pass processing.

## Related tools

- **MALTopic** (Multi-agent LLM framework whose Topic Deduplication Agent (Agent 3) performs the overlap merge on the topic list produced by its Topic Modeling Agent) — yash91sharma/MALTopic
- **openai-python** (Python SDK providing the LLM chat completion calls through which the deduplication agent compares and merges topic labels) — openai/openai-python
- **BERTopic** (Baseline topic modeling method against which MALTopic's deduplicated, refined topic output is evaluated for coherence, diversity, and interpretability)

## Examples

```
from openai import OpenAI; client = OpenAI(); client.chat.completions.create(model="gpt-5.5", messages=[{"role":"user","content":"Merge duplicate and overlapping topics: <topic_list>"}])
```

## Evaluation signals

- Every entry in the output deduplicated_topics list is distinct — no duplicate or semantically overlapping topics remain
- Refined topics retain human-readable labels with enhanced contextual relevance, not machine-generated index strings
- The refined topic list supports improved topic coherence, diversity, and interpretability relative to LDA and BERTopic on the survey dataset
- The input to the step is exactly the topic list from the Topic Modeling Agent (Agent 2), consumed only when response volume exceeded single-pass capacity

## Limitations

- No numeric deduplication threshold is stated in the source — acceptance of the merge is qualitative, so results may vary between LLM runs
- The deduplication agent is only required when the number of survey responses exceeds Agent 2's capacity for single-pass processing; otherwise it is an extra refinement step
- Merging quality depends on the LLM's semantic comparison of labels/keywords; subtle topical distinctions may be collapsed or overlooked

## Evidence

- [other] Agent 3 performs deduplication on Agent 2's topic list: [evidence span withheld — anchor status: not_found]
