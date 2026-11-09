---
name: cosine-similarity-computation
description: Use when consecutive per-second dense captions must be compared semantically
  to decide temporal segment boundaries — e.g., replicating StreamingCoT's DSF segmentation
  with cosine similarity threshold 0.9, or fusing similar adjacent text segments.
when_to_use_negative:
- The comparison target is not temporally adjacent caption text — e.g., clustering
  captions across different videos or documents, where the LIFO stack rule and consecutive-pair
  comparison S(t-1,t) do not apply.
- Input is raw video frames or audio rather than per-second dense caption text; the
  skill presupposes captions a sentence embedding model can encode on a fixed per-second
  grid.
- Segment boundaries must reflect visual scene cuts instead of caption semantics —
  DSF boundaries only coincide with caption-similarity drops below the 0.9 threshold.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: StreamingCoT construction toolkit
  role: Hosts the DSF adaptive temporal segmentation implementation that encodes per-second
    dense captions and computes consecutive-caption cosine similarity for segment
    fusion
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:229d186a5ac7327338625ae2ec252cfb9fee576cf2e5d992ff05939cacddf7c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - StreamingCoT construction toolkit
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cosine-similarity-computation

## Summary

Computes cosine similarity between embeddings of consecutive per-second dense captions to drive the StreamingCoT Dynamic Semantic Fusion (DSF) adaptive temporal segmentation, merging adjacent captions into temporally-dependent semantic segments at threshold theta=0.9.

## When to use

Use when consecutive per-second dense captions must be compared semantically to decide temporal segment boundaries — e.g., replicating StreamingCoT's DSF segmentation with cosine similarity threshold 0.9, or fusing similar adjacent text segments.

## When NOT to use

- The comparison target is not temporally adjacent caption text — e.g., clustering captions across different videos or documents, where the LIFO stack rule and consecutive-pair comparison S(t-1,t) do not apply.
- Input is raw video frames or audio rather than per-second dense caption text; the skill presupposes captions a sentence embedding model can encode on a fixed per-second grid.
- Segment boundaries must reflect visual scene cuts instead of caption semantics — DSF boundaries only coincide with caption-similarity drops below the 0.9 threshold.

## Inputs

- Per-second dense caption sequences for sample videos from StreamingCoT annotation files
- Sentence embedding vector for each per-second caption (produced by a sentence embedding model)

## Outputs

- Semantic segments Seg_i with [start, end] second-resolution boundaries, written to dsf_segments
- Consecutive-caption cosine similarity scores S(t-1,t) along the video timeline

## How to apply

Load the per-second dense caption sequence for a video from the StreamingCoT annotation files and encode each caption into a semantic embedding vector with a sentence embedding model. Compute the cosine similarity S(t-1,t) between each pair of consecutive captions along the video timeline. Apply the DSF decision rule: when S(t-1,t) >= theta=0.9, merge timestamp t into the currently open segment; otherwise pop and emit the completed segment (LIFO stack rule) and push t as the start timestamp of a new segment. Iterate over the full timeline and write the resulting segments Seg_i with [start, end] second-resolution boundaries to dsf_segments. The 0.9 threshold keeps fusion conservative — only near-identical consecutive captions merge — so segment boundaries align with semantic state changes that downstream dynamic QA and multimodal CoT reasoning can reference.

## Related tools

- **StreamingCoT construction toolkit** (Hosts the DSF adaptive temporal segmentation implementation that encodes per-second dense captions and computes consecutive-caption cosine similarity for segment fusion) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
from sklearn.metrics.pairwise import cosine_similarity; S = cosine_similarity(E[:-1], E[1:])  # S[i] = S(t-1,t) for consecutive per-second caption embeddings
```

## Evaluation signals

- Emitted segments are contiguous and jointly cover the entire caption timeline (no gaps or overlaps between consecutive Seg_i).
- Every segment boundary coincides with a consecutive-caption cosine similarity S(t-1,t) below the 0.9 threshold.
- All adjacent caption pairs inside a segment have S(t-1,t) >= 0.9; nothing merges below threshold.
- Segment boundaries are second-resolution integers matching the per-second caption grid, and each record carries a [start, end] pair.
- Manual expert review (per the source card's validation) confirms each emitted segment reads as one coherent semantic unit.

## Limitations

- No specific sentence embedding model is named in the paper or README, so reproductions must choose an encoder; similarity scores and boundary placement depend on that choice.
- theta=0.9 is a fixed threshold — borderline consecutive captions just below 0.9 will split segments even when semantically continuous.
- The StreamingCoT repository provides no changelog ('No changelog found.'), so toolkit behavior may differ from the described pipeline.

## Evidence

- [methods] Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF): "Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF)"
- [abstract] generates per-second dense descriptions and constructs temporally-dependent semantic segments through similarity fusion: "generates per-second dense descriptions and constructs temporally-dependent semantic segments through similarity fusion"
- [readme] Per-second dense captions aligned with visual content: "Per-second dense captions aligned with visual content"
- [methods] Per-second captioning → Dynamic segmentation → Context-aware narration: "Per-second captioning → Dynamic segmentation → Context-aware narration"
