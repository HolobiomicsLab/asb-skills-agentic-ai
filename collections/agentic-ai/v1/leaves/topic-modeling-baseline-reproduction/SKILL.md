---
name: topic-modeling-baseline-reproduction
description: 'Use when re-running the LDA and BERTopic baselines of the MALTopic survey
  dataset: free-text survey responses must be preprocessed identically and the resulting
  topic lists must be validated against deposited topics.'
when_to_use_negative:
- Your goal is the MALTopic multi-agent enrichment pipeline itself (enrichment, topic
  modeling, deduplication agents), not the LDA/BERTopic baselines it is compared against
- Input text is not survey responses concatenated from structured plus free-text fields,
  so comparability with the published Tables II, III, V, VI cannot be established
edam_operation: http://edamontology.org/operation_3432
tools:
- name: BERTopic
  role: Trained on the same preprocessed concatenated survey responses to produce
    the BERTopic baseline topics for both free-text fields
- name: MALTopic
  role: The framework against whose topic coherence, diversity, and interpretability
    the reproduced LDA and BERTopic baselines are compared
  repo: https://github.com/yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:7664cc9b6c9daa9dbc7f388940cdb34a29e2dfe0af3f65841058d297cad9d19e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# topic-modeling-baseline-reproduction

## Summary

Reproduce LDA and BERTopic baselines on concatenated survey responses (structured answers joined with free-text) so their topic lists are directly comparable to MALTopic's outputs, as in the MALTopic paper's Section IV baselines.

## When to use

Use when re-running the LDA and BERTopic baselines of the MALTopic survey dataset: free-text survey responses must be preprocessed identically and the resulting topic lists must be validated against deposited topics.

## When NOT to use

- Your goal is the MALTopic multi-agent enrichment pipeline itself (enrichment, topic modeling, deduplication agents), not the LDA/BERTopic baselines it is compared against
- Input text is not survey responses concatenated from structured plus free-text fields, so comparability with the published Tables II, III, V, VI cannot be established

## Inputs

- Survey dataset with per-respondent structured answers and free-text response fields
- Deposited baseline topic lists [16] for validation

## Outputs

- baseline_topics.csv containing best-iteration LDA and BERTopic topic lists for both free-text fields

## How to apply

Load the survey dataset and concatenate each respondent's structured answers with their free-text responses into single text responses, mirroring the Section IV baselines. Apply the LDA preprocessing pipeline to both free-text fields: lowercasing, punctuation and whitespace removal, stop-word elimination, and lemmatization. Train LDA on the preprocessed text of each free-text field, sweeping over iterations and keeping the best-iteration model whose topics correspond to the entries of Tables II and III (or V and VI). Train BERTopic on the same preprocessed concatenated responses for both free-text fields. Finally, write the best-iteration LDA and BERTopic topic lists for both fields to baseline_topics.csv and compare against the deposited topic lists [16].

## Related tools

- **BERTopic** (Trained on the same preprocessed concatenated survey responses to produce the BERTopic baseline topics for both free-text fields)
- **MALTopic** (The framework against whose topic coherence, diversity, and interpretability the reproduced LDA and BERTopic baselines are compared) — https://github.com/yash91sharma/MALTopic

## Evaluation signals

- Reproduced baseline topic lists match the deposited topic lists [16]
- Reproduced topics match those shown in Tables II and III (LDA) and Tables V and VI
- Both free-text fields were preprocessed with the identical pipeline: lowercasing, punctuation and whitespace removal, stop-word elimination, lemmatization
- LDA model selection is via a sweep over iterations with the best-iteration model retained, not a default single run
- baseline_topics.csv contains topic lists for both free-text fields from both LDA and BERTopic

## Limitations

- Comparability holds only if the exact concatenation of structured answers with free-text responses, as done for the Section IV baselines, is replicated
- LDA results depend on the iteration sweep; stopping the sweep early or skipping best-iteration selection can yield topics that no longer match Tables II, III, V, VI
- Findings of MALTopic's superiority are specific to the survey dataset; the baselines alone do not generalize that comparison

## Evidence

- [methods] Baseline output and validation target: [evidence span withheld — anchor status: not_found]
