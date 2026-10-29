---
name: topic-word-diversity-measurement
description: Use when you have topic word lists from multiple topic models plus the
  underlying survey response corpus and need to quantify which model yields more coherent,
  diverse, and distinct topics.
when_to_use_negative:
- When you only have a single topic model's output and no baseline (LDA/BERTopic)
  to compare against — the skill is framed as a comparative evaluation
- When the input is structured/tabular survey data rather than free-text responses;
  the metrics assume top word lists from textual topics
- When topic quality must be judged by human interpretability alone rather than automated
  corpus-based metrics
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: MALTopic
  role: Topic modeling framework whose generated topic word lists and document-topic
    proportions are evaluated with the four metrics
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose topic word lists are compared against MALTopic
    using the same metrics
- name: openai-python
  role: OpenAI Python API library used to access the LLM back-end powering MALTopic's
    multi-agent (enrichment, topic modeling, deduplication) pipeline
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:9ed87c1738f5022d0095836b91b35a25c742eb188306eb3f7ef0b40ac2524761
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# topic-word-diversity-measurement

## Summary

A skill for evaluating topic models (MALTopic, LDA, BERTopic) on free-text survey data using four complementary metrics: PMI-based word coherence, word diversity (unique-to-total top-word ratio), embedding-based inter-topic similarity, and document coverage at theta=0.1.

## When to use

Use when you have topic word lists from multiple topic models plus the underlying survey response corpus and need to quantify which model yields more coherent, diverse, and distinct topics.

## When NOT to use

- When you only have a single topic model's output and no baseline (LDA/BERTopic) to compare against — the skill is framed as a comparative evaluation
- When the input is structured/tabular survey data rather than free-text responses; the metrics assume top word lists from textual topics
- When topic quality must be judged by human interpretability alone rather than automated corpus-based metrics

## Inputs

- Topic word lists (top words per topic) from MALTopic, LDA, and BERTopic for each free-text survey field
- Underlying survey response corpus for computing word co-occurrence counts
- Document-topic proportion matrices (theta assignments) from each model for document coverage

## Outputs

- topic_metrics.csv containing four metrics (word coherence, word diversity, topic similarity, document coverage) for each of three models across two survey fields
- Grouped bar charts reproducing the Figs. 4-5 metric comparisons

## How to apply

Load the top topic words produced by each model (MALTopic, LDA, BERTopic) for each free-text survey field, together with the response corpus used for word co-occurrence statistics. Compute PMI-based topic coherence per topic by aggregating pairwise positive pointwise mutual information of the top topic words over corpus co-occurrence counts, then average per model. Compute word diversity per model as the ratio of unique words to total words across the top words of all topics. Compute topic similarity by embedding each topic's label or word set and averaging pairwise cosine similarities, and compute document coverage with theta=0.1 by counting documents whose maximum topic proportion exceeds the threshold. Assemble the resulting 4 metrics x 3 models x 2 fields into topic_metrics.csv and render grouped bar charts to compare models.

## Related tools

- **MALTopic** (Topic modeling framework whose generated topic word lists and document-topic proportions are evaluated with the four metrics) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose topic word lists are compared against MALTopic using the same metrics)
- **openai-python** (OpenAI Python API library used to access the LLM back-end powering MALTopic's multi-agent (enrichment, topic modeling, deduplication) pipeline) — https://github.com/openai/openai-python

## Examples

```
diversity = len(set(top_words_flat)) / len(top_words_flat)
```

## Evaluation signals

- The computed metric profile reproduces the reported qualitative pattern: MALTopic increases word coherence and word diversity relative to LDA and BERTopic
- Average inter-topic similarity (mean pairwise cosine similarity of embedded topic sets) is lower for MALTopic than for LDA and BERTopic
- Word diversity values fall in the valid range (0, 1], equal to unique top words divided by total top words across all topics
- Document coverage counts are consistent across models when computed with the same theta=0.1 threshold
- topic_metrics.csv contains all 4 metrics x 3 models x 2 survey fields with no missing cells

## Limitations

- PMI coherence depends on corpus co-occurrence statistics, so results are sensitive to the size and preprocessing of the survey response corpus
- Embedding-based topic similarity depends on the choice of embedding model for topic labels/word sets
- The reported validation is a qualitative pattern (improved coherence/diversity, reduced similarity) on a single survey dataset, not absolute numeric thresholds

## Evidence

- [readme] OpenAI library as LLM back-end access: [evidence span withheld — anchor status: external_doc]
