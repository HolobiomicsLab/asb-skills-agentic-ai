---
name: baseline-topic-list-comparison
description: Use when you have generated topic lists (labels plus representative keywords)
  from MALTopic and at least one baseline (LDA or BERTopic) on a survey dataset and
  need to test whether the multi-agent framework splits themes the baselines merge.
when_to_use_negative:
- The goal is a quantitative coherence/diversity score comparison (C_v, NPMI, topic
  diversity) rather than qualitative topic-level distinctness — use the paper's quantitative
  comparative analysis instead.
- Only one technique's topic list is available; cross-technique separation cannot
  be assessed without LDA and/or BERTopic topic lists.
- The input documents are not survey responses with structured data enrichment available,
  since MALTopic's separation advantage relies on the enrichment agent enhancing textual
  responses.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MALTopic
  role: Source of the generated topic lists under evaluation; its multi-agent pipeline
    (enrichment, topic modeling, deduplication agents) produces the topics whose distinctness
    is tested against baselines
  repo: yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose topic list is compared against MALTopic's to check
    whether it fails to separate the two job-related topics
- name: openai-python
  role: Python client for the OpenAI REST API used to invoke the LLM agents in the
    MALTopic framework that generate the topic lists being compared
  repo: openai/openai-python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:e9c8c189119c7733fee8d639f6940c5dff44d3ff134411bf20140d35c312a879
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# baseline-topic-list-comparison

## Summary

Qualitatively verifies that MALTopic's topic list separates semantically distinct themes (e.g., job-market entry challenges vs. job-displacement concerns) that baseline LDA and BERTopic topic lists conflate, using sentence-embedding cosine similarity and agglomerative hierarchical clustering.

## When to use

Use when you have generated topic lists (labels plus representative keywords) from MALTopic and at least one baseline (LDA or BERTopic) on a survey dataset and need to test whether the multi-agent framework splits themes the baselines merge.

## When NOT to use

- The goal is a quantitative coherence/diversity score comparison (C_v, NPMI, topic diversity) rather than qualitative topic-level distinctness — use the paper's quantitative comparative analysis instead.
- Only one technique's topic list is available; cross-technique separation cannot be assessed without LDA and/or BERTopic topic lists.
- The input documents are not survey responses with structured data enrichment available, since MALTopic's separation advantage relies on the enrichment agent enhancing textual responses.

## Inputs

- Generated topic lists (topic labels plus representative keywords) for MALTopic, LDA, and BERTopic from the survey-dataset pipeline outputs of repo yash91sharma/MALTopic
- Sentence-transformer model for encoding topic labels and keywords

## Outputs

- topic_similarity_matrix.csv (pairwise cosine-similarity matrix across all topics from the three techniques)
- Agglomerative cluster assignments for embedded topics (cosine distance linkage)
- topic_separation_report.csv (cluster membership per technique, within- and between-group similarity scores, cross-technique comparison)

## How to apply

Load the generated topic lists for MALTopic, LDA, and BERTopic from the survey-dataset pipeline outputs of the yash91sharma/MALTopic repo. Encode every topic (label plus representative keywords) into dense sentence embeddings with a sentence-transformer model, then compute the pairwise cosine-similarity matrix across all topics from the three techniques and save it as topic_similarity_matrix.csv. Apply agglomerative hierarchical clustering with cosine distance on the embedded topics to group semantically related topics across techniques. Identify job-related clusters and check whether MALTopic's topics split into two distinct groups (entry-challenge vs. displacement concerns) while LDA and BERTopic topics collapse into one mixed cluster. Summarize cluster membership per technique and within/between-group similarity scores in topic_separation_report.csv.

## Related tools

- **MALTopic** (Source of the generated topic lists under evaluation; its multi-agent pipeline (enrichment, topic modeling, deduplication agents) produces the topics whose distinctness is tested against baselines) — yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose topic list is compared against MALTopic's to check whether it fails to separate the two job-related topics)
- **openai-python** (Python client for the OpenAI REST API used to invoke the LLM agents in the MALTopic framework that generate the topic lists being compared) — openai/openai-python

## Examples

```
from sentence_transformers import SentenceTransformer, util; E = SentenceTransformer('all-MiniLM-L6-v2').encode(topic_texts); S = util.cos_sim(E, E)
```

## Evaluation signals

- Agglomerative clustering with cosine distance reproduces two distinct job-related topic groups for MALTopic (entry-challenge vs. displacement concerns).
- LDA and BERTopic job-related topics collapse into one mixed cluster rather than two separated groups, matching the paper's Section VI.B / Table IV claim.
- topic_similarity_matrix.csv contains pairwise cosine similarities for every topic from all three techniques (no missing entries in the matrix).
- Within-group cosine similarity for each MALTopic job-related group exceeds between-group similarity to the other job-related group.
- topic_separation_report.csv records cluster membership per technique and explicitly flags agreement or disagreement with the paper's reported interpretability advantage.

## Limitations

- The claim is qualitative and dataset-specific (survey responses on job-related themes); a clustering result that does not reproduce two distinct MALTopic job-related groups counts as disagreement with the Section VI.B / Table IV claim, not a method failure.
- Results depend on the chosen sentence-transformer embedding and agglomerative clustering linkage/distance (cosine); different encoders may yield different cluster structures.
- MALTopic's separation advantage is tied to its multi-agent design (enrichment, topic modeling, deduplication agents) over LLM APIs, so reproducibility depends on the LLM backend available via the OpenAI client.

## Evidence

- [abstract] MALTopic improves coherence/diversity/interpretability over baselines: "Comparative analysis on a survey dataset demonstrates that MALTopic significantly improves topic coherence, diversity, and interpretability compared to LDA and BERTopic."
- [abstract] Multi-agent pipeline that generates the topic lists: "an enrichment agent leverages structured data to enhance textual responses, a topic modeling agent extracts latent themes, and a deduplication agent refines the results"
- [other] Embedding and clustering workflow for the comparison: [evidence span withheld — anchor status: not_found]
- [readme] OpenAI client used to drive the LLM agents: [evidence span withheld — anchor status: external_doc]
