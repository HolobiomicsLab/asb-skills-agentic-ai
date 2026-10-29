---
name: embedding-cosine-topic-similarity-evaluation
description: Use when you have top-word lists from multiple topic models (e.g., MALTopic
  vs LDA vs BERTopic) plus the underlying free-text survey response corpus, and you
  need to quantify coherence, diversity, and inter-topic redundancy.
when_to_use_negative:
- Input is not a set of topic word lists plus a raw text corpus — PMI coherence requires
  corpus-level word co-occurrence counts and cannot be computed from topic lists alone.
- The analysis goal is topic model training/enrichment (the enrichment, topic-modeling,
  and deduplication agents) rather than post-hoc comparison of already-produced topic
  outputs.
- Document-topic proportions are unavailable, making the theta=0.1 Document Coverage
  metric impossible to compute reliably.
tools:
- name: MALTopic
  role: Source of the topic word lists whose coherence, diversity, and inter-topic
    similarity are being scored against LDA and BERTopic baselines
  repo: yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model compared against MALTopic in the four-metric evaluation
- name: openai-python
  role: OpenAI Python client library used to access the OpenAI API for embedding topic
    labels/word sets prior to cosine similarity computation
  repo: openai/openai-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:34fec7b76d0b06277888840669ab9c4cd01add1cef801beafe1e135e2b6fa292
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# embedding-cosine-topic-similarity-evaluation

## Summary

An evaluation skill for the MALTopic paper that scores topic models (MALTopic, LDA, BERTopic) on survey free-text fields using PMI-based word coherence, word diversity, embedding-based inter-topic cosine similarity, and document coverage (theta=0.1), reproducing the Figs. 4-5 comparison showing MALTopic yields more coherent, more diverse, less redundant topics.

## When to use

Use when you have top-word lists from multiple topic models (e.g., MALTopic vs LDA vs BERTopic) plus the underlying free-text survey response corpus, and you need to quantify coherence, diversity, and inter-topic redundancy.

## When NOT to use

- Input is not a set of topic word lists plus a raw text corpus — PMI coherence requires corpus-level word co-occurrence counts and cannot be computed from topic lists alone.
- The analysis goal is topic model training/enrichment (the enrichment, topic-modeling, and deduplication agents) rather than post-hoc comparison of already-produced topic outputs.
- Document-topic proportions are unavailable, making the theta=0.1 Document Coverage metric impossible to compute reliably.

## Inputs

- Topic word lists for each topic from MALTopic, LDA, and BERTopic (two free-text survey fields)
- Underlying free-text survey response corpus for word co-occurrence statistics
- Per-document topic proportion matrices (document-topic distributions) for coverage computation

## Outputs

- topic_metrics.csv with four metrics (word coherence, word diversity, topic similarity, document coverage) x three models x two survey fields
- Grouped bar charts reproducing the Figs. 4-5 metric comparisons

## How to apply

Load the topic word lists produced by MALTopic, LDA, and BERTopic for each of the two free-text survey fields, along with the response corpus used for word co-occurrence statistics. Compute PMI-based topic coherence per topic by aggregating pairwise positive pointwise mutual information of the top topic words over corpus co-occurrence counts, then average per model. Compute Word Diversity as the ratio of unique words to total words across the top words of all topics. Embed each topic's label/word set (e.g., via an OpenAI embeddings client built on the openai-python library) and average pairwise cosine similarities between topics per model to obtain Topic Similarity. Finally, compute Document Coverage with theta=0.1 by counting documents whose maximum topic proportion exceeds the threshold, and assemble the four metrics x three models x two fields into topic_metrics.csv for grouped bar charts mirroring Figs. 4 and 5.

## Related tools

- **MALTopic** (Source of the topic word lists whose coherence, diversity, and inter-topic similarity are being scored against LDA and BERTopic baselines) — yash91sharma/MALTopic
- **BERTopic** (Baseline topic model compared against MALTopic in the four-metric evaluation)
- **openai-python** (OpenAI Python client library used to access the OpenAI API for embedding topic labels/word sets prior to cosine similarity computation) — openai/openai-python

## Examples

```
from sklearn.metrics.pairwise import cosine_similarity; import numpy as np; S = cosine_similarity(topic_embeddings); mean_inter_topic_sim = np.mean(S[np.triu_indices_from(S, 1)])
```

## Evaluation signals

- The computed metric profile reproduces the reported qualitative pattern: MALTopic shows increased word coherence and word diversity relative to LDA and BERTopic.
- Mean pairwise inter-topic cosine similarity for MALTopic is lower than for LDA and BERTopic on both free-text survey fields.
- Word Diversity values fall in [0, 1] as a unique-words/total-words ratio, and PMI coherence values are finite and positive where co-occurrence exists.
- topic_metrics.csv contains exactly 4 metrics x 3 models x 2 fields rows with no missing cells before charting.
- Document Coverage is computed with the fixed threshold theta=0.1 (documents whose maximum topic proportion exceeds theta).

## Limitations

- Embedding-based Topic Similarity depends on the choice of embedding model and on how each topic's label/word set is embedded, so values are not directly comparable across embedding choices.
- PMI-based coherence requires the full response corpus for co-occurrence statistics; sparse fields can yield unstable PMI estimates.
- Validation is qualitative (matching the reported pattern of improved coherence/diversity and reduced similarity), not an exact numeric replication.
- The openai-python library requires Python 3.10+ and an OpenAI API key for embedding calls.

## Evidence

- [readme] OpenAI Python client used for embedding access: [evidence span withheld — anchor status: external_doc]
