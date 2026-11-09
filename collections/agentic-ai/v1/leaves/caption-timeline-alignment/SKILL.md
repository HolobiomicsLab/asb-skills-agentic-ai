---
name: caption-timeline-alignment
description: Use when you have per-second dense caption sequences over a video timeline
  and must group them into contiguous semantic segments whose boundaries coincide
  with drops in consecutive-caption cosine similarity (theta = 0.9).
when_to_use_negative:
- The video or captions are not indexed per second (the DSF rule operates on consecutive
  second-resolution caption pairs)
- You need question-answer pairs or reasoning-chain (CoT) annotations rather than
  temporally-dependent semantic segments — those come from the downstream Dynamic
  QA Generation and Multimodal CoT Synthesis stages
- Captions are already pre-segmented (e.g., human-authored narrative segments), in
  which case re-fusing at theta = 0.9 would alter existing boundaries
edam_operation: http://edamontology.org/operation_2990
tools:
- name: StreamingCoT toolkit
  role: Source of the annotation files containing per-second dense captions and the
    DSF segmentation stage of the hierarchical captioning pipeline
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: FFmpeg
  role: Video handling underlying the per-second caption timeline that DSF segments
- name: OpenCV
  role: Computer-vision library cited by the paper for the video processing side of
    the annotation pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:9d33e8152b464cb30f6dc8d3fe504f6f47df919a1e063e64dda007b4fe27b01a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - FFmpeg
  - OpenCV
  - StreamingCoT toolkit
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# caption-timeline-alignment

## Summary

Reconstructs the Dynamic Semantic Fusion (DSF) adaptive temporal segmentation used by StreamingCoT: per-second dense captions are encoded as embeddings, consecutive-caption cosine similarity drives a LIFO-stack grouping of the timeline into temporally-dependent semantic segments with second-resolution [start, end] boundaries.

## When to use

Use when you have per-second dense caption sequences over a video timeline and must group them into contiguous semantic segments whose boundaries coincide with drops in consecutive-caption cosine similarity (theta = 0.9).

## When NOT to use

- The video or captions are not indexed per second (the DSF rule operates on consecutive second-resolution caption pairs)
- You need question-answer pairs or reasoning-chain (CoT) annotations rather than temporally-dependent semantic segments — those come from the downstream Dynamic QA Generation and Multimodal CoT Synthesis stages
- Captions are already pre-segmented (e.g., human-authored narrative segments), in which case re-fusing at theta = 0.9 would alter existing boundaries

## Inputs

- Per-second dense caption sequences for sample videos (from StreamingCoT annotation files)
- Sentence embedding model for encoding each per-second caption
- Similarity threshold theta = 0.9

## Outputs

- Semantic segments Seg_i with second-resolution [start, end] boundaries written to dsf_segments
- Validation report confirming contiguity, full-timeline coverage, and threshold-coincident boundaries

## How to apply

Load the per-second dense caption sequences for the sample videos from the StreamingCoT annotation files, then encode each caption into a semantic embedding vector with a sentence embedding model. Compute the cosine similarity S(t-1,t) between each pair of consecutive captions along the video timeline; when S(t-1,t) >= theta = 0.9, merge timestamp t into the currently open segment, otherwise trigger the LIFO stack rule: pop and emit the completed segment and push t as the start timestamp of a new segment. Iterate over the full timeline so the emitted segments Seg_i are contiguous and jointly cover the entire caption timeline, then write them to dsf_segments. Validation follows the pipeline's expert-review approach: every segment boundary must coincide with a consecutive-caption cosine similarity below the 0.9 threshold, mirroring the paper's 'similarity fusion' construction of temporally-dependent semantic segments.

## Related tools

- **StreamingCoT toolkit** (Source of the annotation files containing per-second dense captions and the DSF segmentation stage of the hierarchical captioning pipeline) — https://github.com/Fleeting-hyh/StreamingCoT
- **FFmpeg** (Video handling underlying the per-second caption timeline that DSF segments)
- **OpenCV** (Computer-vision library cited by the paper for the video processing side of the annotation pipeline)

## Examples

```
sim = 1 - cosine(emb[t-1], emb[t]); seg.append(t) if sim >= 0.9 else (emit(stack.pop()), stack.push(new_segment(t)))
```

## Evaluation signals

- Emitted segments are contiguous with no gaps between consecutive [start, end] boundaries
- Segments jointly cover the entire per-second caption timeline (first segment starts at second 0, last ends at the final caption second)
- Every segment boundary coincides with a consecutive-caption cosine similarity strictly below the theta = 0.9 threshold
- All internal (non-boundary) seconds have consecutive-caption cosine similarity >= 0.9
- Output file dsf_segments exists and each Seg_i record carries second-resolution start and end values

## Limitations

- The 0.9 cosine-similarity cutoff and LIFO pop rule are dataset-specific choices from StreamingCoT; other corpora may require recalibration of theta
- Boundaries are constrained to second resolution and to caption-change points, so within-second semantic shifts cannot be separated
- Quality of segmentation depends on the chosen sentence embedding model; the paper does not specify a particular encoder
- No changelog is available for the StreamingCoT repository, so toolkit-level behavior may drift from the published algorithm

## Evidence

- [methods] Adaptive Temporal Segmentation stage uses a Dynamic Semantic Fusion (DSF) algorithm that employs a Last-In-First-Out (LIFO) stack: [evidence span withheld — anchor status: not_found]
- [abstract] per-second dense descriptions and similarity-fusion of temporally-dependent semantic segments: "Our framework first establishes a dynamic hierarchical annotation architecture that generates per-second dense descriptions and constructs temporally-dependent semantic segments through similarity"
- [readme] Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF): "Adaptive temporal segmentation via Dynamic Semantic Fusion (DSF)"
- [readme] Per-second dense captions aligned with visual content: "- **Per-second dense captions** aligned with visual content"
