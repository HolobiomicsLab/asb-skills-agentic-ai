---
name: temporal-caption-embedding
description: Use when per-second dense caption sequences must be grouped into temporally-dependent
  semantic segments whose boundaries coincide with drops in consecutive-caption semantic
  similarity.
when_to_use_negative:
- Captions are not aligned at per-second temporal resolution — the merge/segment decision
  assumes one caption per timeline second
- The goal is keyframe object grounding or QA generation rather than temporal segmentation
  — those are separate StreamingCoT pipeline stages
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: StreamingCoT toolkit
  role: Provides the DSF adaptive temporal segmentation stage that consumes the per-second
    dense captions and emits temporally-dependent semantic segments
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:13ca7540c1279eb7b77c8d6ae38f9aef7376ac319cf533ef50ecd9fd58a701be
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - StreamingCoT toolkit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# temporal-caption-embedding

## Summary

Encodes per-second dense video captions into sentence embedding vectors and uses consecutive-caption cosine similarity (threshold theta=0.9) to drive the Dynamic Semantic Fusion (DSF) adaptive temporal segmentation of the StreamingCoT annotation pipeline.

## When to use

Use when per-second dense caption sequences must be grouped into temporally-dependent semantic segments whose boundaries coincide with drops in consecutive-caption semantic similarity.

## When NOT to use

- Captions are not aligned at per-second temporal resolution — the merge/segment decision assumes one caption per timeline second
- The goal is keyframe object grounding or QA generation rather than temporal segmentation — those are separate StreamingCoT pipeline stages

## Inputs

- Per-second dense caption sequences for sample videos from StreamingCoT annotation files
- Sentence embedding model (sentence encoder for caption semantics)

## Outputs

- dsf_segments file with semantic segments Seg_i carrying [start, end] second-resolution boundaries
- Consecutive-caption cosine similarity values S(t-1,t) along the video timeline

## How to apply

Load the per-second dense captions from the StreamingCoT annotation files and encode each caption into a semantic embedding vector with a sentence embedding model. Compute the cosine similarity S(t-1,t) between each pair of consecutive captions along the video timeline. When S(t-1,t) >= theta=0.9, merge timestamp t into the currently open segment; otherwise apply the LIFO stack rule: pop and emit the completed segment and push timestamp t as the start of a new segment. Iterate over the full timeline to output semantic segments Seg_i with [start, end] second-resolution boundaries and write them to dsf_segments. The 0.9 threshold reflects the paper's 'similarity fusion' of temporally-dependent semantic segments in its dynamic hierarchical annotation architecture.

## Related tools

- **StreamingCoT toolkit** (Provides the DSF adaptive temporal segmentation stage that consumes the per-second dense captions and emits temporally-dependent semantic segments) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
seg = [caps[0].start]; for t in range(1, len(caps)): if cosine(emb[t-1], emb[t]) >= 0.9: seg[-1].end = t; else: emit(seg.pop()); seg.append(Segment(start=t))
```

## Evaluation signals

- Emitted segments are contiguous — no gaps between consecutive Seg_i [start, end] boundaries
- Segments jointly cover the entire caption timeline
- Every segment boundary coincides with a consecutive-caption cosine similarity below the 0.9 threshold
- All internal consecutive-caption similarities within a segment are >= theta=0.9
- Segment boundaries are at second resolution, matching the per-second caption granularity

## Limitations

- Fixed theta=0.9 threshold — captions with subtly drifting topics may be over-merged or over-split depending on embedding model behavior
- Segmentation quality depends on the chosen sentence embedding model; the article does not specify which encoder is used
- Boundary decisions are local (only consecutive-caption similarity), so no long-range semantic coherence constraint is applied

## Evidence

- [other] The pipeline's Adaptive Temporal Segmentation stage uses a Dynamic Semantic Fusion (DSF) algorithm that employs a Last-In-First-Out (LIFO) stack to group per-second captions into semantic segments.: [evidence span withheld — anchor status: not_found]
