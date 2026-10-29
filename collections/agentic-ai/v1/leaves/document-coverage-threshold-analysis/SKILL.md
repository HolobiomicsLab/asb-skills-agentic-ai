---
name: document-coverage-threshold-analysis
description: Use when comparing topic models on survey free-text data and you need
  to measure what share of documents each model assigns to topics with sufficient
  strength (max topic proportion > theta, e.g. theta=0.1).
when_to_use_negative:
- Input is a topic-word list only, with no per-document topic proportion distributions
  available (coherence/diversity metrics apply instead)
- You need embedding-based inter-topic similarity comparison rather than document-level
  coverage (use the topic similarity skill)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: MALTopic
  role: Source of per-document topic proportion distributions for the proposed multi-agent
    LLM topic modeling framework
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose document-topic distributions are compared against
    MALTopic and LDA on document coverage
- name: openai-python
  role: OpenAI API client used to power the multi-agent LLM steps (enrichment, topic
    modeling, deduplication) that produce MALTopic's topic assignments
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:dc76d63e368e090b23efe248c562ea983060c0ef420775c100d895636066f593
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# document-coverage-threshold-analysis

## Summary

Computes Document Coverage for topic models (MALTopic, LDA, BERTopic) by counting the fraction of survey documents whose maximum topic proportion exceeds a fixed theta threshold (theta=0.1), enabling quantitative comparison of how much of the corpus each model actually explains.

## When to use

Use when comparing topic models on survey free-text data and you need to measure what share of documents each model assigns to topics with sufficient strength (max topic proportion > theta, e.g. theta=0.1).

## When NOT to use

- Input is a topic-word list only, with no per-document topic proportion distributions available (coherence/diversity metrics apply instead)
- You need embedding-based inter-topic similarity comparison rather than document-level coverage (use the topic similarity skill)

## Inputs

- Per-document topic proportion matrices (document-topic distributions) from MALTopic, LDA, and BERTopic for each free-text survey field
- Theta threshold parameter (theta=0.1 in the paper's evaluation)
- Underlying survey response corpus for the two free-text fields

## Outputs

- Document Coverage score per model per field (fraction of documents with max topic proportion > theta)
- Rows in topic_metrics.csv combining coverage, word coherence, word diversity, and topic similarity
- Grouped bar charts reproducing Figures 4-5 comparisons

## How to apply

For each model (MALTopic, LDA, BERTopic) fitted on the two free-text survey fields, obtain per-document topic proportion distributions (e.g. theta/document-topic matrices). For each document take the maximum topic proportion, count documents where this maximum exceeds the theta threshold (theta=0.1 as used in the MALTopic evaluation), and divide by the total number of documents to get Document Coverage. Assemble coverage alongside word coherence, word diversity, and topic similarity metrics into topic_metrics.csv and render grouped bar charts reproducing the Figures 4-5 comparisons. Validation: coverage should be interpreted together with the qualitative pattern that MALTopic increases coherence and diversity while reducing inter-topic similarity relative to LDA and BERTopic.

## Related tools

- **MALTopic** (Source of per-document topic proportion distributions for the proposed multi-agent LLM topic modeling framework) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose document-topic distributions are compared against MALTopic and LDA on document coverage)
- **openai-python** (OpenAI API client used to power the multi-agent LLM steps (enrichment, topic modeling, deduplication) that produce MALTopic's topic assignments) — https://github.com/openai/openai-python

## Examples

```
import pandas as pd; coverage = (doc_topic.max(axis=1) > 0.1).mean()  # theta=0.1 per MALTopic eval
```

## Evaluation signals

- Coverage is a fraction in [0, 1]; count of documents exceeding theta divided by total document count must be recomputable from the document-topic matrix
- Using theta=0.1 as the paper specifies — verify the threshold is fixed across all three models and both survey fields for a fair comparison
- The combined topic_metrics.csv must reproduce the reported pattern: MALTopic increases word coherence and diversity while reducing inter-topic similarity vs LDA and BERTopic
- Grouped bar chart must show four metrics x three models x two fields, matching the layout of Figures 4 and 5

## Limitations

- Document Coverage depends on the choice of theta; the paper uses theta=0.1, and results at other thresholds are not reported
- Coverage is evaluated on a survey dataset with two free-text fields; generalization to other corpora is not demonstrated
- Metrics are computed from top topic words and document proportions, so coverage figures depend on the upstream quality of the enrichment, topic modeling, and deduplication agents

## Evidence

- [abstract] MALTopic improves topic coherence, diversity, and interpretability compared to LDA and BERTopic on a survey dataset: "Comparative analysis on a survey dataset demonstrates that MALTopic significantly improves topic coherence, diversity, and interpretability compared to LDA and BERTopic."
- [other] reproduce Figs 4-5 metric comparison: [evidence span withheld — anchor status: not_found]
