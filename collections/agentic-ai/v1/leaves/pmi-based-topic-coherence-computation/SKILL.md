---
name: pmi-based-topic-coherence-computation
description: Use when comparing topic models on survey/free-text corpora and you need
  word coherence, word diversity, inter-topic similarity, and document coverage metrics
  from top-word lists.
when_to_use_negative:
- Input is raw survey text rather than already-generated topic word lists and document-topic
  distributions from a fitted topic model
- The goal is qualitative interpretability review of topic labels rather than the
  quantitative coherence/diversity/similarity/coverage comparison
- The corpus lacks the free-text survey fields the co-occurrence statistics must be
  computed over
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: MALTopic
  role: Multi-agent LLM topic modeling framework whose generated topics are evaluated
    with the coherence, diversity, similarity, and coverage metrics
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model against which MALTopic's metric profile is compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:b30c8cc22403699505a1a88bd6182cd260b816178893db384fb9b4ca624c976e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pmi-based-topic-coherence-computation

## Summary

Computes PMI-based word coherence, word diversity, embedding-based inter-topic similarity, and document coverage (theta=0.1) to quantitatively compare topic models such as MALTopic, LDA, and BERTopic on free-text survey data, reproducing the Fig. 4-5 metric comparison.

## When to use

Use when comparing topic models on survey/free-text corpora and you need word coherence, word diversity, inter-topic similarity, and document coverage metrics from top-word lists.

## When NOT to use

- Input is raw survey text rather than already-generated topic word lists and document-topic distributions from a fitted topic model
- The goal is qualitative interpretability review of topic labels rather than the quantitative coherence/diversity/similarity/coverage comparison
- The corpus lacks the free-text survey fields the co-occurrence statistics must be computed over

## Inputs

- Topic word lists (top words per topic) for MALTopic, LDA, and BERTopic, per survey field
- Underlying free-text survey response corpus for word co-occurrence statistics
- Document-topic proportion distributions (for document coverage at theta=0.1)
- Embedding model for topic label/word-set embeddings (topic similarity)

## Outputs

- topic_metrics.csv with word coherence, word diversity, topic similarity, and document coverage per model per field
- Grouped bar charts reproducing the Fig. 4-5 metric comparisons

## How to apply

Load the topic word lists produced by each model (MALTopic, LDA, BERTopic) for each free-text survey field, together with the underlying response corpus used to build word co-occurrence counts. For every topic, aggregate pairwise positive pointwise mutual information of its top words over the corpus co-occurrence statistics and average per model to obtain word coherence. Compute word diversity per model as the ratio of unique words to total words across the top words of all topics. For topic similarity, embed each topic's label/word set and average the pairwise cosine similarities between topics; compute document coverage with theta=0.1 by counting documents whose maximum topic proportion exceeds the threshold. Assemble the four metrics x three models x two fields into topic_metrics.csv and render grouped bar charts reproducing Figs. 4 and 5.

## Related tools

- **MALTopic** (Multi-agent LLM topic modeling framework whose generated topics are evaluated with the coherence, diversity, similarity, and coverage metrics) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model against which MALTopic's metric profile is compared)

## Evaluation signals

- Computed metric profile must reproduce the reported qualitative pattern: MALTopic increases word coherence and word diversity while reducing average inter-topic similarity relative to LDA and BERTopic
- Word diversity values lie in (0, 1] as a ratio of unique words to total words across all top topic words
- Document coverage counts only documents whose maximum topic proportion exceeds the theta=0.1 threshold
- topic_metrics.csv contains all four metrics x three models x two survey fields with no missing cells

## Limitations

- Metric conclusions hold on the paper's survey dataset and two free-text fields; generalization to other corpora is not established by this evidence
- PMI coherence depends on the specific response corpus used for co-occurrence counts, so results are corpus-sensitive

## Evidence

- [other] PMI coherence computation: [evidence span withheld — anchor status: not_found]
- [other] document coverage threshold: [evidence span withheld — anchor status: not_found]
