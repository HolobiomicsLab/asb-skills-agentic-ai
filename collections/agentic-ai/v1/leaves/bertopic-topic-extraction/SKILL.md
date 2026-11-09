---
name: bertopic-topic-extraction
description: Use when a survey dataset's concatenated respondent free-text fields
  need BERTopic baseline topics to benchmark against MALTopic, after the LDA preprocessing
  pipeline (lowercasing, punctuation/whitespace removal, stop-word removal, lemmatization)
  has been applied.
when_to_use_negative:
- You are reproducing MALTopic itself - its enrichment, topic-modeling, and deduplication
  agents integrate structured survey data that the plain BERTopic baseline does not
- The survey responses have not yet been concatenated and LDA-preprocessed - BERTopic
  on raw free text will not match the deposited baselines [16]
- The goal is contextually enriched, human-readable topics built from structured answers
  - per the abstract, only MALTopic's multi-agent approach delivers that over BERTopic
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0203
tools:
- name: BERTopic
  role: Topic model trained on the preprocessed concatenated survey responses to produce
    the baseline topics for each free-text field
- name: MALTopic
  role: Framework whose reported significant gains in topic coherence, diversity,
    and interpretability over BERTopic define the comparison this baseline must enable
  repo: https://github.com/yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:682c74c40dd8d054e43045445f81fa2a2dc41eb0edbcc23b39d959e7bf97d280
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bertopic-topic-extraction

## Summary

Reproduces the MALTopic paper's BERTopic baseline: BERTopic is trained on the same LDA-preprocessed, concatenated survey responses used for the LDA baselines, producing topic lists per free-text field that must match the deposited topics [16] so MALTopic's coherence, diversity, and interpretability gains can be benchmarked.

## When to use

Use when a survey dataset's concatenated respondent free-text fields need BERTopic baseline topics to benchmark against MALTopic, after the LDA preprocessing pipeline (lowercasing, punctuation/whitespace removal, stop-word removal, lemmatization) has been applied.

## When NOT to use

- You are reproducing MALTopic itself - its enrichment, topic-modeling, and deduplication agents integrate structured survey data that the plain BERTopic baseline does not
- The survey responses have not yet been concatenated and LDA-preprocessed - BERTopic on raw free text will not match the deposited baselines [16]
- The goal is contextually enriched, human-readable topics built from structured answers - per the abstract, only MALTopic's multi-agent approach delivers that over BERTopic

## Inputs

- Survey dataset with per-respondent structured answers and free-text responses, concatenated into single text responses for each free-text field
- Preprocessed text corpora for both free-text fields (lowercased, punctuation/whitespace removed, stop words eliminated, lemmatized)
- Deposited baseline topic lists [16] and topic tables (Tables II, III, V, VI) used as validation references

## Outputs

- BERTopic baseline topic list for each free-text field
- baseline_topics.csv with the best-iteration LDA and BERTopic topic lists for both free-text fields

## How to apply

Concatenate each respondent's structured answers with their free-text responses into single text responses per free-text field, mirroring the Section IV baseline setup. Pass both fields through the LDA preprocessing pipeline - lowercasing, punctuation and extraneous-whitespace removal, stop-word elimination, and lemmatization - so BERTopic sees the identical corpus as LDA and the baselines stay comparable to MALTopic. Train BERTopic on the preprocessed concatenated responses of each free-text field to obtain the BERTopic baseline topics. Write the best-iteration LDA and BERTopic topic lists for both fields to baseline_topics.csv and compare them against the deposited topic lists [16] and the entries of Tables II, III, V, VI. The rationale is that MALTopic claims significant improvements in topic coherence, diversity, and interpretability over LDA and BERTopic on this survey dataset, so the baseline must reproduce the deposited topics exactly for that comparison to be valid.

## Related tools

- **BERTopic** (Topic model trained on the preprocessed concatenated survey responses to produce the baseline topics for each free-text field)
- **MALTopic** (Framework whose reported significant gains in topic coherence, diversity, and interpretability over BERTopic define the comparison this baseline must enable) — https://github.com/yash91sharma/MALTopic

## Examples

```
from bertopic import BERTopic; topics, probs = BERTopic().fit_transform(preprocessed_concat)
```

## Evaluation signals

- Reproduced BERTopic topic lists for both free-text fields match the deposited topic lists [16]
- Reproduced baseline topics correspond to the entries shown in Tables II, III, V, and VI
- baseline_topics.csv contains best-iteration topic lists for both LDA and BERTopic across both free-text fields
- BERTopic was trained on the identical preprocessed concatenated responses used for LDA (lowercased, punctuation/whitespace removed, stop words eliminated, lemmatized)
- Baseline coherence and diversity metrics for BERTopic are available so MALTopic's claimed significant improvements can be measured against them

## Limitations

- The BERTopic baseline does not leverage structured survey data, so its topics lack the contextual relevance MALTopic obtains via its enrichment agent
- Comparability depends on the exact LDA preprocessing pipeline; deviating from lowercasing, punctuation/whitespace removal, stop-word elimination, or lemmatization breaks the match with deposited topics [16]
- Evidence is limited to one survey dataset with two free-text fields, on which MALTopic significantly outperforms BERTopic in coherence, diversity, and interpretability

## Evidence

- [abstract] Why the BERTopic baseline is reproduced for this survey dataset: "Comparative analysis on a survey dataset demonstrates that MALTopic significantly improves topic coherence, diversity, and interpretability compared to LDA and BERTopic."
- [abstract] What the baseline lacks relative to MALTopic: "By integrating structured data and employing a multi-agent approach, MALTopic generates human-readable topics with enhanced contextual relevance"
- [methods] Preprocessing pipeline shared by LDA and BERTopic baselines: [evidence span withheld — anchor status: not_found]
- [methods] How BERTopic training is done for the baseline: [evidence span withheld — anchor status: not_found]
- [results] Success criterion for the reproduced baseline: [evidence span withheld — anchor status: not_found]
