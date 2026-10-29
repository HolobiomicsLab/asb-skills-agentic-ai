---
name: video-temporal-boundary-detection
description: Use when a video annotation pipeline already has per-second dense captions
  and you need to fuse them into semantic segments with temporally-dependent boundaries
  (second resolution) via similarity-based adaptive temporal segmentation.
when_to_use_negative:
- Input videos lack per-second dense captions (the algorithm operates on per-second
  caption sequences, not raw frames or shot cuts)
- You need visual shot/cut boundary detection rather than semantic caption-based segmentation
- Captions are at coarser-than-second resolution, since the DSF stack rule assumes
  one caption per timestamp along the timeline
tools:
- name: StreamingCoT construction toolkit
  role: Repository toolkit implementing the hierarchical annotation pipeline, including
    the Dynamic Semantic Fusion adaptive temporal segmentation stage
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:742f653b534d0d5ecb23d554914f2f69fed5c41f47923f43d5e8102021d33973
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - StreamingCoT construction toolkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# video-temporal-boundary-detection

## Summary

Reconstructs the Dynamic Semantic Fusion (DSF) algorithm from the StreamingCoT pipeline, which groups per-second dense captions into temporally-dependent semantic segments using a LIFO stack and a consecutive-caption cosine similarity threshold of theta=0.9. It produces second-resolution [start, end] segment boundaries that serve as the basis for downstream dynamic QA and reasoning-chain annotation.

## When to use

Use when a video annotation pipeline already has per-second dense captions and you need to fuse them into semantic segments with temporally-dependent boundaries (second resolution) via similarity-based adaptive temporal segmentation.

## When NOT to use

- Input videos lack per-second dense captions (the algorithm operates on per-second caption sequences, not raw frames or shot cuts)
- You need visual shot/cut boundary detection rather than semantic caption-based segmentation
- Captions are at coarser-than-second resolution, since the DSF stack rule assumes one caption per timestamp along the timeline

## Inputs

- Per-second dense caption sequences from StreamingCoT annotation files (one caption per second of video)
- Sentence embedding model used to vectorize each caption

## Outputs

- Semantic segments Seg_i with second-resolution [start, end] boundaries written to dsf_segments
- Segment set that is contiguous and jointly covers the entire caption timeline

## How to apply

Load the per-second dense caption sequences for each sample video from the StreamingCoT annotation files and encode every caption into a semantic embedding vector with a sentence embedding model. Compute the cosine similarity S(t-1,t) between each pair of consecutive captions along the video timeline. When S(t-1,t) >= theta=0.9, merge timestamp t into the currently open segment; otherwise apply the LIFO stack rule — pop and emit the completed segment, and push t as the start timestamp of a new segment. Iterate over the full timeline to output semantic segments Seg_i with their [start, end] second-resolution boundaries and write them to dsf_segments. The rationale is that a fixed 0.9 similarity cutoff mirrors the paper's similarity-fusion design, producing boundaries only where consecutive second-level captions diverge semantically rather than at arbitrary time intervals.

## Related tools

- **StreamingCoT construction toolkit** (Repository toolkit implementing the hierarchical annotation pipeline, including the Dynamic Semantic Fusion adaptive temporal segmentation stage) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
from sklearn.metrics.pairwise import cosine_similarity; S = cosine_similarity(emb[t-1:t], emb[t:t+1])[0][0]; merge if S >= 0.9 else stack.pop() and emit segment
```

## Evaluation signals

- Emitted segments are contiguous (no gaps between consecutive [start, end] boundaries)
- Segments jointly cover the entire caption timeline from the first to the last per-second caption
- Every segment boundary coincides with a consecutive-caption cosine similarity below the 0.9 threshold
- Within every emitted segment, all consecutive-caption similarities are >= 0.9
- Boundaries fall on integer second timestamps matching the caption timeline resolution

## Limitations

- Boundary quality depends entirely on the quality and per-second density of the input captions; the article's per-second dense captions are generated by its hierarchical captioning stage
- The 0.9 cosine threshold is fixed in the workflow; tuning it would change segment granularity and would no longer match the published pipeline
- The paper does not specify which sentence embedding model is used, so reproduction requires selecting a model and validating against the threshold-based invariants

## Evidence
