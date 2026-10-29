---
name: survey-free-text-topic-analysis
description: Use when raw survey data with structured answers plus free-text responses
  must be turned into baseline topic lists (LDA/BERTopic) comparable to a reference
  topic-modeling method on the same dataset.
when_to_use_negative:
- The goal is the MALTopic method itself (enrichment agent + structured-data integration
  + deduplication agent), not the LDA/BERTopic baselines it is compared against
- Input text is already preprocessed and de-duplicated, so re-running lowercasing,
  stop-word removal, and lemmatization would alter token distributions relative to
  the deposited baselines
edam_operation: http://edamontology.org/operation_0306
tools:
- name: BERTopic
  role: Baseline topic model trained on the same preprocessed concatenated survey
    responses as LDA to make results comparable to MALTopic
- name: MALTopic
  role: Multi-agent LLM topic-modeling framework whose outputs (coherence, diversity,
    interpretability) the reproduced LDA/BERTopic baselines are benchmarked against
  repo: yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:d845ff9e234f4b4f66e4bac00fa5390dd250c6415b1ad7e87f1ac5665b784625
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# survey-free-text-topic-analysis

## Summary

Reproduce LDA and BERTopic baselines on concatenated survey responses (structured answers joined with free-text fields) so their topics can be compared against the MALTopic multi-agent LLM framework on coherence, diversity, and interpretability.

## When to use

Use when raw survey data with structured answers plus free-text responses must be turned into baseline topic lists (LDA/BERTopic) comparable to a reference topic-modeling method on the same dataset.

## When NOT to use

- The goal is the MALTopic method itself (enrichment agent + structured-data integration + deduplication agent), not the LDA/BERTopic baselines it is compared against
- Input text is already preprocessed and de-duplicated, so re-running lowercasing, stop-word removal, and lemmatization would alter token distributions relative to the deposited baselines

## Inputs

- Survey dataset containing per-respondent structured answers and free-text response fields
- Deposited reference topic lists (citation [16]) for validation

## Outputs

- baseline_topics.csv containing best-iteration LDA and BERTopic topic lists for both free-text fields

## How to apply

First concatenate each respondent's structured answers with their free-text responses into single text documents, as done for the Section IV baselines. Apply the LDA preprocessing pipeline to the free-text fields: lowercasing, removal of punctuation and extraneous whitespace, stop-word elimination, and word lemmatization. Train LDA on the preprocessed text of each free-text field, sweeping over iterations and keeping the best-iteration model, then train BERTopic on the same preprocessed concatenated responses for both fields. Write the best-iteration LDA and BERTopic topic lists for both fields to baseline_topics.csv and validate them by diffing against the deposited topic lists [16] and the topics shown in Tables II, III, V, VI of the paper.

## Related tools

- **BERTopic** (Baseline topic model trained on the same preprocessed concatenated survey responses as LDA to make results comparable to MALTopic)
- **MALTopic** (Multi-agent LLM topic-modeling framework whose outputs (coherence, diversity, interpretability) the reproduced LDA/BERTopic baselines are benchmarked against) — yash91sharma/MALTopic

## Evaluation signals

- Reproduced best-iteration LDA topic lists match the deposited topic lists [16]
- Reproduced BERTopic topics match those in the deposited lists and the paper's topic tables
- Baseline topics match the corresponding entries of Tables II, III, V, and VI
- Both free-text fields are processed identically (same lowercasing/punctuation/whitespace/stop-word/lemmatization pipeline) so baselines remain comparable to MALTopic

## Limitations

- LDA results depend on the iteration sweep: only the best-iteration model is retained, so a different sweep range may not reproduce the deposited topics
- Comparability holds only if the exact preprocessing (lowercasing, punctuation and whitespace removal, stop-word elimination, lemmatization) and the concatenation of structured answers with free text are replicated; deviations break the baseline-to-MALTopic comparison

## Evidence

- [full_text] Baseline workflow: [evidence span withheld — anchor status: not_found]
