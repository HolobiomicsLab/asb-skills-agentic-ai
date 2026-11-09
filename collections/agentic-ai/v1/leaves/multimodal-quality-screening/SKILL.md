---
name: multimodal-quality-screening
description: Use when building a curated video corpus from YouTube API candidates
  and you need to enforce combined social, textual, visual-resolution, motion, and
  aesthetic quality thresholds before annotation.
when_to_use_negative:
- Candidate videos are not sourced from YouTube or lack the metadata (interactions,
  resolution, captions) needed to score every filter criterion
- The goal is annotation (per-second captioning, QA generation, CoT synthesis) on
  videos that have already passed quality screening — apply the downstream StreamingCoT
  pipeline stages instead
- You only need geographic balancing (stratified geographic sampling) without quality-based
  rejection; that is a separate pipeline step preceding this filter
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: YouTube API
  role: Source of candidate video metadata (interaction counts, resolution, captions)
    on which the multimodal quality filters are evaluated
- name: FFmpeg
  role: Video processing for resolution and stream-related quality checks during screening
- name: OpenCV
  role: Computer-vision library supporting motion dynamics analysis and aesthetic
    assessment of candidate videos
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:f41559477fd0e9ab29a880e6c72d7f826bc51cc79f41fe92162fe9f6be87652d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FFmpeg
  - OpenCV
  - YouTube API
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multimodal-quality-screening

## Summary

A multi-criterion filtering step in the StreamingCoT video collection pipeline that retains only candidate YouTube videos passing social validation (>5,000 aggregated interactions), lexical density constraints, HD resolution (≥720p), motion dynamics analysis, and aesthetic scoring (≥7/10), ensuring a high-quality short-video corpus for streaming VideoQA.

## When to use

Use when building a curated video corpus from YouTube API candidates and you need to enforce combined social, textual, visual-resolution, motion, and aesthetic quality thresholds before annotation.

## When NOT to use

- Candidate videos are not sourced from YouTube or lack the metadata (interactions, resolution, captions) needed to score every filter criterion
- The goal is annotation (per-second captioning, QA generation, CoT synthesis) on videos that have already passed quality screening — apply the downstream StreamingCoT pipeline stages instead
- You only need geographic balancing (stratified geographic sampling) without quality-based rejection; that is a separate pipeline step preceding this filter

## Inputs

- Video metadata records (one per YouTube video) fetched via the YouTube API, including social interaction counts (likes, comments, shares), resolution, captions/transcripts, and aesthetic/motion scores
- Candidate video streams (≤60 seconds) after geographic balancing via stratified geographic sampling

## Outputs

- Filtered set of high-quality short videos (≤60s, ≥720p, >5,000 aggregated interactions, aesthetic score ≥7/10) admitted to the Hierarchical Captioning stage
- Per-video filter decision records (pass/fail per criterion, e.g. interaction_total, aesthetic_score columns)

## How to apply

Start from video metadata records retrieved via the YouTube API (the pipeline order is YouTube API → Geographic balancing → Multimodal quality screening). Apply the filter cascade: (1) social validation — compute the aggregated social interaction total per video and retain only videos exceeding 5,000 interactions; (2) lexical density constraints on caption/transcript text; (3) HD resolution check requiring ≥720p; (4) motion dynamics analysis of the video stream; (5) aesthetic scoring with a ≥7/10 cutoff. Only videos passing all criteria proceed to per-second hierarchical captioning; the filters are deliberately stacked so that low-engagement, low-resolution, static, or visually poor videos are excluded before expensive annotation. Videos are short-form (≤60 seconds) and the corpus is globally balanced via stratified geographic sampling.

## Related tools

- **YouTube API** (Source of candidate video metadata (interaction counts, resolution, captions) on which the multimodal quality filters are evaluated)
- **FFmpeg** (Video processing for resolution and stream-related quality checks during screening)
- **OpenCV** (Computer-vision library supporting motion dynamics analysis and aesthetic assessment of candidate videos)

## Examples

```
df['interaction_total'] = df['likes'] + df['comments'] + df['shares']; passed = df[(df.interaction_total > 5000) & (df.resolution >= 720) & (df.aesthetic_score >= 7)]; passed.to_csv('social_validation_passed.csv', index=False)
```

## Evaluation signals

- Every retained video has aggregated social interactions strictly greater than 5,000 (sum of likes, comments, shares); any record at or below 5,000 is excluded
- Every retained video has resolution ≥720p (HD) and an aesthetic score ≥7/10
- Retained videos satisfy lexical density constraints and pass motion dynamics analysis (i.e., are not static/near-static streams)
- All retained videos are ≤60 seconds and were drawn from the stratified geographic sampling stage
- The filtered corpus is comparable in scale and composition to the published 5,745 high-quality short videos

## Limitations

- Filter thresholds (>5,000 interactions, ≥720p, ≥7/10 aesthetics) are dataset-specific choices for StreamingCoT's short-video corpus; they may discard suitable videos for other domains
- All videos are sourced from YouTube and remain subject to original content creators' rights; users must comply with YouTube's Terms of Service, which constrains reuse of the filtered corpus
- Social validation via aggregate interaction counts is a proxy for quality that may bias the corpus toward already-popular content

## Evidence
