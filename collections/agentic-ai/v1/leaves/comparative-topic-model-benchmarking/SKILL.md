---
name: comparative-topic-model-benchmarking
description: Use when you have topic word lists from multiple topic models (e.g.,
  MALTopic, LDA, BERTopic) plus the underlying response corpus, and need to quantitatively
  compare coherence, diversity, similarity, and coverage per model.
when_to_use_negative:
- Input is a single model's topic output with no baseline (LDA/BERTopic) to compare
  against, so no comparative benchmark can be formed
- The corpus or topic word lists are unavailable, making PMI co-occurrence statistics
  and word-diversity ratios impossible to compute
- The goal is evaluating MALTopic's enrichment/deduplication agents rather than comparing
  final topic quality metrics across models
tools:
- name: MALTopic
  role: Multi-agent LLM topic modeling framework whose topic word lists and document-topic
    distributions are one of the three benchmarked models; reference implementation
    in yash91sharma/MALTopic
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose topics are benchmarked against MALTopic and LDA
    on the four metrics
- name: openai-python
  role: OpenAI Python API library used to invoke the LLM agents (enrichment, topic
    modeling, deduplication) behind MALTopic's topic generation
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:1269676877d993ffcedcf6ceeb313c7f6c4673376d7e6ed3b857928e91e2e194
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
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# comparative-topic-model-benchmarking

## Summary

Benchmark topic quality across MALTopic, LDA, and BERTopic on free-text survey fields using four metrics: PMI-based word coherence, word diversity, embedding-based inter-topic similarity, and document coverage, reproducing the Figs. 4-5 comparison showing MALTopic's improved coherence and diversity with reduced inter-topic similarity.

## When to use

Use when you have topic word lists from multiple topic models (e.g., MALTopic, LDA, BERTopic) plus the underlying response corpus, and need to quantitatively compare coherence, diversity, similarity, and coverage per model.

## When NOT to use

- Input is a single model's topic output with no baseline (LDA/BERTopic) to compare against, so no comparative benchmark can be formed
- The corpus or topic word lists are unavailable, making PMI co-occurrence statistics and word-diversity ratios impossible to compute
- The goal is evaluating MALTopic's enrichment/deduplication agents rather than comparing final topic quality metrics across models

## Inputs

- Topic word lists from MALTopic, LDA, and BERTopic for each of two free-text survey fields
- Underlying survey response corpus used for word co-occurrence statistics
- Document-topic proportion matrices (topic distributions) for coverage computation

## Outputs

- topic_metrics.csv with four metrics (word coherence, word diversity, topic similarity, document coverage) x three models x two fields
- Grouped bar charts reproducing the Figs. 4-5 metric comparisons

## How to apply

Load the topic word lists produced by MALTopic, LDA, and BERTopic for each free-text survey field, together with the underlying response corpus for word co-occurrence statistics. Compute PMI-based topic coherence for every topic by aggregating pairwise positive pointwise mutual information of top topic words over corpus co-occurrence counts, then average per model. Compute Word Diversity per model as the ratio of unique words to total words across the top words of all topics, and Topic Similarity by embedding each topic's label/word set and averaging pairwise cosine similarities between topics. Compute Document Coverage with theta=0.1, counting documents whose maximum topic proportion exceeds the theta threshold. Assemble the four metrics x three models x two fields into topic_metrics.csv and render grouped bar charts reproducing Figs. 4 and 5.

## Related tools

- **MALTopic** (Multi-agent LLM topic modeling framework whose topic word lists and document-topic distributions are one of the three benchmarked models; reference implementation in yash91sharma/MALTopic) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose topics are benchmarked against MALTopic and LDA on the four metrics)
- **openai-python** (OpenAI Python API library used to invoke the LLM agents (enrichment, topic modeling, deduplication) behind MALTopic's topic generation) — https://github.com/openai/openai-python

## Examples

```
pip install openai; from openai import OpenAI; client = OpenAI()
```

## Evaluation signals

- Computed metric profile must reproduce the reported qualitative pattern: MALTopic increases word coherence and word diversity while reducing average inter-topic similarity relative to LDA and BERTopic
- Word diversity per model equals the ratio of unique words to total words across the top words of all topics (value in (0, 1])
- Document coverage uses theta=0.1 consistently across models, counting documents whose maximum topic proportion exceeds theta
- topic_metrics.csv contains the full 4 metrics x 3 models x 2 fields grid with no missing cells
- Topic similarity values are averages of pairwise cosine similarities over topic embeddings, bounded in [-1, 1]

## Limitations

- Validation is qualitative: the check is the reproduced pattern (MALTopic higher coherence/diversity, lower similarity), not exact numeric equivalence to the published figures
- PMI coherence depends on the specific survey response corpus used for co-occurrence counts; results are dataset-specific to the survey data
- Coverage results depend on the chosen theta threshold (0.1); other thresholds may change the ranking

## Evidence

- [abstract] Comparative analysis on a survey dataset demonstrates that MALTopic significantly improves topic coherence, diversity, and interpretability compared to LDA and BERTopic.: "Comparative analysis on a survey dataset demonstrates that MALTopic significantly improves topic coherence, diversity, and interpretability compared to LDA and BERTopic."
- [abstract] Multi-agent pipeline that produces the topics being benchmarked.: "an enrichment agent leverages structured data to enhance textual responses, a topic modeling agent extracts latent themes, and a deduplication agent refines the results"
- [readme] OpenAI client library grounding the LLM invocation behind MALTopic.: [evidence span withheld — anchor status: external_doc]
