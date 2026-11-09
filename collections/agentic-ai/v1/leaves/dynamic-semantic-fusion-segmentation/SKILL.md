---
name: dynamic-semantic-fusion-segmentation
description: Use when per-second dense captions must be grouped into temporally-dependent
  semantic segments for hierarchical video annotation, and segment boundaries should
  coincide with semantic similarity drops between consecutive captions.
when_to_use_negative:
- Input is raw video or key frames rather than per-second dense caption sequences
  — DSF operates on caption text embeddings, not pixels
- Segmentation must allow overlapping or non-contiguous segments — DSF by construction
  emits segments that jointly and contiguously cover the caption timeline
- A single fixed-granularity shot detector or scene cut detector is desired instead
  of caption-semantic-driven adaptive segmentation
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: StreamingCoT construction toolkit
  role: Provides the annotation files (per-second dense captions) that DSF consumes
    and the pipeline stage 'Dynamic segmentation' the algorithm implements
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: Sentence embedding model
  role: Encodes each per-second caption into a semantic embedding vector for consecutive-pair
    cosine similarity computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:f0bab9d3a85d2abb71f5c360ed9ba2b128446bbb969ddaf647094daf41f7cb78
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - Sentence embedding model
  - StreamingCoT construction toolkit
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dynamic-semantic-fusion-segmentation

## Summary

Reconstructs the Dynamic Semantic Fusion (DSF) adaptive temporal segmentation algorithm from StreamingCoT, which uses per-second dense caption embeddings, consecutive-caption cosine similarity, and a LIFO stack to group captions into contiguous semantic segments with second-resolution [start, end] boundaries.

## When to use

Use when per-second dense captions must be grouped into temporally-dependent semantic segments for hierarchical video annotation, and segment boundaries should coincide with semantic similarity drops between consecutive captions.

## When NOT to use

- Input is raw video or key frames rather than per-second dense caption sequences — DSF operates on caption text embeddings, not pixels
- Segmentation must allow overlapping or non-contiguous segments — DSF by construction emits segments that jointly and contiguously cover the caption timeline
- A single fixed-granularity shot detector or scene cut detector is desired instead of caption-semantic-driven adaptive segmentation

## Inputs

- Per-second dense caption sequences for sample videos from the StreamingCoT annotation files
- Sentence embedding model for encoding each per-second caption into a semantic vector
- Similarity threshold theta = 0.9

## Outputs

- Semantic segments Seg_i with [start, end] second-resolution boundaries, written to dsf_segments

## How to apply

Load the per-second dense caption sequences for each sample video from the StreamingCoT annotation files and encode each caption into a semantic embedding vector with a sentence embedding model. Compute the cosine similarity S(t-1,t) between each pair of consecutive captions along the video timeline. When S(t-1,t) >= theta = 0.9, merge timestamp t into the currently open segment; otherwise apply the LIFO stack rule: pop and emit the completed segment and push timestamp t as the start of a new segment. Iterate over the full timeline to emit segments Seg_i with [start, end] second-resolution boundaries and write them to dsf_segments. The rationale is that StreamingCoT's dynamic hierarchical annotation constructs 'temporally-dependent semantic segments through similarity fusion' on top of per-second dense descriptions, so the threshold and stack rule reproduce that exact segmentation stage.

## Related tools

- **StreamingCoT construction toolkit** (Provides the annotation files (per-second dense captions) that DSF consumes and the pipeline stage 'Dynamic segmentation' the algorithm implements) — https://github.com/Fleeting-hyh/StreamingCoT
- **Sentence embedding model** (Encodes each per-second caption into a semantic embedding vector for consecutive-pair cosine similarity computation)

## Examples

```
python dsf_segment.py --captions streamingcot/per_second_captions/VIDEO_ID.jsonl --theta 0.9 --out dsf_segments/VIDEO_ID.json
```

## Evaluation signals

- Every emitted segment is contiguous and the union of segments jointly covers the entire per-second caption timeline
- Every segment boundary coincides with a consecutive-caption cosine similarity below the 0.9 threshold
- All internal consecutive-caption pairs within a segment have cosine similarity >= 0.9
- Segment boundaries are at second resolution, matching the per-second caption timestamps of the source captions
- Rerunning the algorithm on the same caption sequence is deterministic and reproduces the same dsf_segments output

## Limitations

- The threshold theta = 0.9 is fixed; captions with gradual semantic drift across many seconds may never trigger a boundary since adjacent pairs each stay above threshold
- Only consecutive-caption similarity is considered, so long-range semantic re-entry (a topic returning after an intervening dissimilar segment) starts a new segment rather than merging with the earlier one
- Quality depends on the per-second dense captions and the sentence embedding model; the anonymous 4open.science mirror and GitHub repo provide no changelog, so algorithm drift between toolkit versions cannot be tracked

## Evidence

- [abstract] generates per-second dense descriptions and constructs temporally-dependent semantic segments through similarity fusion: "generates per-second dense descriptions and constructs temporally-dependent semantic segments through similarity fusion"
- [methods] Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF): "Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF)"
- [readme] Per-second captioning → Dynamic segmentation → Context-aware narration: "Per-second captioning → Dynamic segmentation → Context-aware narration"
- [readme] Per-second dense captions aligned with visual content; Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF): "Per-second dense captions aligned with visual content"
