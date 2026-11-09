---
name: topic-embedding-similarity-computation
description: Use when comparing topic lists produced by different topic-modeling techniques
  (e.g., MALTopic vs. LDA vs. BERTopic on the same survey dataset) and you need a
  semantic, embedding-based separation analysis rather than token-overlap metrics
  like coherence or diversity.
when_to_use_negative:
- You only need standard intrinsic quality metrics (topic coherence, diversity) already
  reported per technique, not cross-technique semantic separation of individual topic
  pairs.
- The input is a single technique's topic list with no comparable topics from other
  methods to align or cluster against.
- Topics are short keyword-only strings without labels or contextual keywords, making
  sentence embeddings of 'label plus representative keywords' unreliable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MALTopic
  role: Source of the framework-generated topic lists whose qualitative distinctness
    (two separate job-related topics) is being tested against the baselines via embedding
    similarity and clustering
  repo: https://github.com/yash91sharma/MALTopic
- name: BERTopic
  role: Baseline topic model whose topic list is embedded and clustered alongside
    MALTopic's to verify the baselines fail to separate the two job-related topics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:552e87ae2cb0f560a21699edf5e08b775a0e88e9c6705ce4b26cfffb166af085
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# topic-embedding-similarity-computation

## Summary

Encodes topic labels plus representative keywords from MALTopic, LDA, and BERTopic into dense sentence embeddings and computes a pairwise cosine-similarity matrix to quantify cross-technique semantic overlap, enabling hierarchical clustering that tests whether MALTopic's job-related topics are qualitatively more distinct than baseline topics.

## When to use

Use when comparing topic lists produced by different topic-modeling techniques (e.g., MALTopic vs. LDA vs. BERTopic on the same survey dataset) and you need a semantic, embedding-based separation analysis rather than token-overlap metrics like coherence or diversity.

## When NOT to use

- You only need standard intrinsic quality metrics (topic coherence, diversity) already reported per technique, not cross-technique semantic separation of individual topic pairs.
- The input is a single technique's topic list with no comparable topics from other methods to align or cluster against.
- Topics are short keyword-only strings without labels or contextual keywords, making sentence embeddings of 'label plus representative keywords' unreliable.

## Inputs

- Generated topic lists (labels + representative keywords) from MALTopic, LDA, and BERTopic outputs of the survey-dataset pipeline (repo yash91sharma/MALTopic)
- Sentence-transformer model for dense topic embeddings
- Agglomerative hierarchical clustering configuration using cosine distance

## Outputs

- topic_similarity_matrix.csv — pairwise cosine-similarity matrix across all topics from the three techniques
- topic_separation_report.csv — cluster membership per technique plus similarity scores within and between job-related groups
- Hierarchical clustering of embedded topics indicating whether job-related topics separate into distinct groups per technique

## How to apply

Load the generated topic lists for MALTopic, LDA, and BERTopic from the survey-dataset pipeline outputs. Encode every topic (label plus representative keywords) into dense sentence embeddings using a sentence-transformer model, then compute the full pairwise cosine-similarity matrix across topics from all three techniques and persist it as topic_similarity_matrix.csv. Apply agglomerative hierarchical clustering with cosine distance on the embedded topics to group semantically related topics across techniques. Inspect the job-related clusters to check whether MALTopic's topics split into two distinct groups (job-market entry challenges for students/inexperienced respondents vs. job-displacement concerns of experienced professionals) while LDA and BERTopic topics collapse into one mixed cluster. Summarize cluster membership per technique and similarity scores within and between job-related groups in topic_separation_report.csv; record agreement or disagreement with the Section VI.B / Table IV interpretability claim.

## Related tools

- **MALTopic** (Source of the framework-generated topic lists whose qualitative distinctness (two separate job-related topics) is being tested against the baselines via embedding similarity and clustering) — https://github.com/yash91sharma/MALTopic
- **BERTopic** (Baseline topic model whose topic list is embedded and clustered alongside MALTopic's to verify the baselines fail to separate the two job-related topics)

## Examples

```
from sentence_transformers import SentenceTransformer; from sklearn.metrics.pairwise import cosine_similarity; S = cosine_similarity(SentenceTransformer('all-MiniLM-L6-v2').encode(topic_labels_and_keywords))
```

## Evaluation signals

- topic_similarity_matrix.csv is square with dimensions equal to the total number of topics across MALTopic, LDA, and BERTopic, with unit diagonal (cosine self-similarity = 1).
- Agglomerative clustering with cosine distance reproduces two distinct job-related topic groups for MALTopic (entry-challenge vs. displacement concerns).
- LDA and BERTopic job-related topics collapse into one mixed cluster rather than separating into the two groups.
- topic_separation_report.csv contains, per technique, cluster membership plus within-group and between-group similarity scores for the job-related clusters.
- The report's conclusion is consistent with the paper's Section VI.B / Table IV interpretability-advantage claim, or explicitly records the disagreement.

## Limitations

- Separation results depend on the chosen sentence-transformer embedding and cosine-distance linkage, so the two-group split for MALTopic may not be robust across embedding models.
- The validation is specific to job-related topics on one survey dataset; other topic domains or datasets may not exhibit the same entry-challenge vs. displacement separation.
- Agreement with the Section VI.B / Table IV claim is a validation criterion, not a guarantee — the report must record disagreement if clustering does not reproduce the paper's separation.

## Evidence

- [methods] Embedding of topic lists from three techniques: [evidence span withheld — anchor status: not_found]
- [methods] Similarity matrix output artifact: [evidence span withheld — anchor status: not_found]
- [results] Expected separation finding: [evidence span withheld — anchor status: not_found]
