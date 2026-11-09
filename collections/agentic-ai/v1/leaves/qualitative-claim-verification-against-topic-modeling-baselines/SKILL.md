---
name: qualitative-claim-verification-against-topic-modeling-baselines
description: Use when a topic-modeling paper claims one method separates semantically
  distinct themes that baseline methods (LDA, BERTopic) merge, and you must independently
  verify that claim from generated topic lists.
when_to_use_negative:
- The claim concerns quantitative metrics (coherence, diversity scores) rather than
  qualitative topic separability — compute the metrics directly instead of clustering
  topic labels.
- Only one technique's topic list is available; cross-technique separation claims
  cannot be tested without LDA and BERTopic outputs on the same survey data.
- Topics are short keyword n-grams too sparse for meaningful sentence embeddings;
  embedding label+keywords may not capture the intended semantics.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: MALTopic
  role: Source of the treatment topic lists whose claimed separation into two distinct
    job-related topics is under verification; multi-agent LLM framework with enrichment,
    topic modeling, and deduplication agents
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose topic list is compared against MALTopic's to test
    whether the two job-related topics are conflated
- name: openai-python
  role: Python client used to access the LLM backend powering MALTopic's multi-agent
    pipeline
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:3cf5a2bc603f8a4e7d86a16aea0e7b79f80e76e4cbec46a42b51b36a1c4d7442
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
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# Qualitative claim verification against topic modeling baselines

## Summary

Reproduces and adjudicates a paper's qualitative interpretability claim — that MALTopic separates two job-related topics (job-market entry challenges vs. job-displacement concerns) that LDA and BERTopic conflate — by embedding all topics, computing cross-technique cosine similarity, and running agglomerative hierarchical clustering.

## When to use

Use when a topic-modeling paper claims one method separates semantically distinct themes that baseline methods (LDA, BERTopic) merge, and you must independently verify that claim from generated topic lists.

## When NOT to use

- The claim concerns quantitative metrics (coherence, diversity scores) rather than qualitative topic separability — compute the metrics directly instead of clustering topic labels.
- Only one technique's topic list is available; cross-technique separation claims cannot be tested without LDA and BERTopic outputs on the same survey data.
- Topics are short keyword n-grams too sparse for meaningful sentence embeddings; embedding label+keywords may not capture the intended semantics.

## Inputs

- Generated topic lists (topic label + representative keywords) from MALTopic on the survey dataset
- Generated topic lists from baseline methods LDA and BERTopic on the same survey dataset
- Sentence-transformer model for dense topic embeddings

## Outputs

- topic_similarity_matrix.csv — pairwise cosine-similarity matrix across all topics from the three techniques
- topic_separation_report.csv — cross-technique comparison with cluster membership per technique and similarity scores within and between job-related groups

## How to apply

Load the generated topic lists (label plus representative keywords) for MALTopic, LDA, and BERTopic from the survey-dataset pipeline outputs in the yash91sharma/MALTopic repository. Encode every topic into dense sentence embeddings with a sentence-transformer model, then compute the full pairwise cosine-similarity matrix across topics from all three techniques. Apply agglomerative hierarchical clustering using cosine distance to group semantically related topics across techniques. The decision point is whether MALTopic's job-related topics split into two distinct clusters (entry-challenge vs. displacement concerns) while LDA and BERTopic topics collapse into one mixed cluster; record cluster membership and within/between-group similarity scores to either support or dispute the paper's Section VI.B / Table IV interpretability claim.

## Related tools

- **MALTopic** (Source of the treatment topic lists whose claimed separation into two distinct job-related topics is under verification; multi-agent LLM framework with enrichment, topic modeling, and deduplication agents) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose topic list is compared against MALTopic's to test whether the two job-related topics are conflated)
- **openai-python** (Python client used to access the LLM backend powering MALTopic's multi-agent pipeline) — https://github.com/openai/openai-python

## Examples

```
from sentence_transformers import SentenceTransformer; from sklearn.metrics.pairwise import cosine_similarity; emb = SentenceTransformer('all-MiniLM-L6-v2').encode(topic_texts); S = cosine_similarity(emb)
```

## Evaluation signals

- The agglomerative clustering (cosine distance) reproduces exactly two distinct job-related topic groups for MALTopic, matching the paper's claim.
- LDA and BERTopic job-related topics collapse into a single mixed cluster rather than separating into the two groups.
- topic_similarity_matrix.csv contains pairwise cosine similarities for every topic from all three techniques with no missing entries.
- topic_separation_report.csv records cluster membership per technique plus within-group and between-group similarity scores for the job-related clusters.
- If clustering contradicts the claim, the report explicitly records the disagreement with the Section VI.B / Table IV claim rather than silently passing.

## Limitations

- Verification depends on qualitative clustering of embedded topic labels/keywords; different sentence-transformer models may yield different cluster structures.
- The claim is dataset-specific (survey dataset) and tied to the two job-related themes (entry challenges for students/inexperienced respondents vs. displacement concerns of experienced professionals); it may not generalize to other corpora.
- The openai-python client requires a valid OPENAI_API_KEY, so reproducing MALTopic's topic lists incurs API access and cost.

## Evidence
