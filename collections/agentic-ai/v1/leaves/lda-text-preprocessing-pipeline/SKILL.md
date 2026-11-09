---
name: lda-text-preprocessing-pipeline
description: Use when reproducing the Section IV LDA/BERTopic baselines on concatenated
  survey responses, or when free-text survey answers must be preprocessed (lowercasing,
  punctuation/whitespace removal, stop words, lemmatization) before topic modeling.
when_to_use_negative:
- The goal is to run MALTopic itself — its enrichment agent consumes structured data
  directly, whereas this skill only prepares the LDA/BERTopic baseline inputs
- The corpus is not survey responses and no deposited baseline topic lists exist to
  validate the preprocessing against
- Text is already normalized (lowercased, stop-word-filtered, lemmatized) and concatenation
  has been performed
edam_operation: http://edamontology.org/operation_0306
tools:
- name: BERTopic
  role: Trained on the same preprocessed concatenated survey responses to produce
    the BERTopic baseline topics for comparison with MALTopic
  repo: yash91sharma/MALTopic
- name: MALTopic
  role: Reference method against which the reproduced LDA and BERTopic baselines are
    compared; baselines mirror its Section IV setup
  repo: yash91sharma/MALTopic
- name: openai-python
  role: OpenAI Python API library associated with the MALTopic repository context
    (LLM access for the multi-agent framework)
  repo: openai/openai-python
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:ab0654221c816014a577e292781468fabad260d08f125758114c08297b5ea106
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lda-text-preprocessing-pipeline

## Summary

Reproduces the LDA baseline text preprocessing used in the MALTopic survey study: concatenating structured and free-text survey answers, then normalizing text (lowercasing, punctuation/whitespace removal, stop-word elimination, lemmatization) so LDA and BERTopic baselines are comparable to MALTopic.

## When to use

Use when reproducing the Section IV LDA/BERTopic baselines on concatenated survey responses, or when free-text survey answers must be preprocessed (lowercasing, punctuation/whitespace removal, stop words, lemmatization) before topic modeling.

## When NOT to use

- The goal is to run MALTopic itself — its enrichment agent consumes structured data directly, whereas this skill only prepares the LDA/BERTopic baseline inputs
- The corpus is not survey responses and no deposited baseline topic lists exist to validate the preprocessing against
- Text is already normalized (lowercased, stop-word-filtered, lemmatized) and concatenation has been performed

## Inputs

- Survey dataset with structured answers and free-text responses per respondent
- Deposited baseline topic lists (reference [16]) for validation

## Outputs

- Preprocessed (lowercased, stop-word-free, lemmatized) concatenated response corpus per free-text field
- baseline_topics.csv with best-iteration LDA and BERTopic topic lists for both free-text fields

## How to apply

First load the survey dataset and concatenate each respondent's structured answers with their free-text responses into single text responses, as done for the Section IV baselines. Apply the LDA preprocessing pipeline to the free-text fields: lowercasing, removal of punctuation and extraneous whitespace, elimination of stop words, and word lemmatization. Train LDA on the preprocessed text of each free-text field, sweeping over iterations and selecting the best-iteration model whose topics correspond to Tables II and III (or V and VI). Train BERTopic on the same preprocessed concatenated responses for both free-text fields so the baselines share identical inputs and are comparable to MALTopic. Finally, write the best-iteration LDA and BERTopic topic lists to baseline_topics.csv and diff them against the deposited topic lists [16].

## Related tools

- **BERTopic** (Trained on the same preprocessed concatenated survey responses to produce the BERTopic baseline topics for comparison with MALTopic) — yash91sharma/MALTopic
- **MALTopic** (Reference method against which the reproduced LDA and BERTopic baselines are compared; baselines mirror its Section IV setup) — yash91sharma/MALTopic
- **openai-python** (OpenAI Python API library associated with the MALTopic repository context (LLM access for the multi-agent framework)) — openai/openai-python

## Evaluation signals

- Reproduced baseline topic lists match the deposited topic lists [16]
- Best-iteration LDA topics match the topics shown in Tables II, III, V, and VI of the paper
- BERTopic is trained on the exact same preprocessed concatenated responses as LDA (shared corpus check)
- Preprocessed text contains no uppercase characters, punctuation, stop words, or unlemmatized tokens
- baseline_topics.csv contains topic lists for both free-text fields from both LDA and BERTopic

## Limitations

- Validation depends on the availability of the deposited topic lists [16]; without them, baseline fidelity cannot be confirmed by diffing
- The preprocessing recipe is specific to the survey dataset used in the MALTopic study; other corpora may require different normalization choices
- LDA requires sweeping over iterations to select the best-iteration model, so results depend on the iteration sweep range

## Evidence

- [methods] concatenation step: [evidence span withheld — anchor status: not_found]
