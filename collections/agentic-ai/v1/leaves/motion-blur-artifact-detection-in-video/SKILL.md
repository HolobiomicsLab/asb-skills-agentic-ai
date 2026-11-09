---
name: motion-blur-artifact-detection-in-video
description: Use when curation of a short-video (≤60 s) corpus requires excluding
  clips with excessive motion blur or near-static content, i.e., per-frame motion
  statistics and an exclusion decision must be derived from optical-flow analysis.
when_to_use_negative:
- Videos are already post-filter high-quality corpus clips that passed the StreamingCoT
  multimodal quality screening — re-filtering adds no information.
- The goal is aesthetic, social, lexical, or resolution screening (e.g., HD ≥720p,
  aesthetic score ≥7/10, >5,000 interactions) — those are separate filter steps in
  the pipeline.
- Analysis targets single images or keyframes rather than temporal frame sequences,
  since the method depends on frame-to-frame flow variance.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3393
tools:
- name: OpenCV
  role: VideoCapture frame decoding and dense optical-flow (Farneback) computation
    of per-frame flow variance
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: FFmpeg
  role: FFmpeg-backed codecs used to decode video frames for analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:2afa082d49aabb462a8a56b0cbba4e6c0d0547e6a215a63c28bf152dbf356082
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - FFmpeg
  - OpenCV
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# motion-blur-artifact-detection-in-video

## Summary

Screens short video candidates for motion-quality defects by computing dense optical-flow variance over 30-second windows, excluding videos where ≥15% of frames show high variance (motion blur) or static segments occupy ≥85% of the window. This implements the 'Motion dynamics analysis' filter in the StreamingCoT multimodal quality-screening pipeline.

## When to use

Use when curation of a short-video (≤60 s) corpus requires excluding clips with excessive motion blur or near-static content, i.e., per-frame motion statistics and an exclusion decision must be derived from optical-flow analysis.

## When NOT to use

- Videos are already post-filter high-quality corpus clips that passed the StreamingCoT multimodal quality screening — re-filtering adds no information.
- The goal is aesthetic, social, lexical, or resolution screening (e.g., HD ≥720p, aesthetic score ≥7/10, >5,000 interactions) — those are separate filter steps in the pipeline.
- Analysis targets single images or keyframes rather than temporal frame sequences, since the method depends on frame-to-frame flow variance.

## Inputs

- Sampled candidate videos (≤60 seconds, YouTube-sourced, MP4 or similar decodable by OpenCV/FFmpeg)

## Outputs

- motion_filter_report.csv with columns: video_id, high_variance_frame_fraction, static_occupancy, excluded, reason
- Per-video motion statistics (per-frame flow-variance scores, high-variance frame fractions, static-segment occupancy per 30-second window)

## How to apply

Load each sampled candidate video (≤60 s, YouTube-sourced) with OpenCV VideoCapture using FFmpeg-backed codecs, then compute dense frame-to-frame optical flow (e.g., Farneback) and derive a per-frame flow-variance score quantifying motion dynamics. Within each 30-second window, mark frames as high-variance (excessive motion blur) or as belonging to static segments, and compute two statistics: the fraction of high-variance frames and the temporal occupancy of static segments. Flag the video for exclusion if ≥15% of its frames are high-variance or if static segments occupy ≥85% of the window, recording the flag reason per video. Write per-video statistics and decisions to motion_filter_report.csv with fields video_id, high_variance_frame_fraction, static_occupancy, excluded, reason; the rationale is to keep only videos with usable temporal dynamics for streaming VideoQA annotation.

## Related tools

- **OpenCV** (VideoCapture frame decoding and dense optical-flow (Farneback) computation of per-frame flow variance) — https://github.com/Fleeting-hyh/StreamingCoT
- **FFmpeg** (FFmpeg-backed codecs used to decode video frames for analysis)

## Examples

```
import cv2; flow = cv2.calcOpticalFlowFarneback(prev_gray, next_gray, None, 0.5, 3, 15, 3, 5, 1.2, 0)
```

## Evaluation signals

- motion_filter_report.csv exists and contains one row per processed video with the required fields (video_id, high_variance_frame_fraction, static_occupancy, excluded, reason).
- Excluded flags are consistent with the thresholds: excluded=true only when high_variance_frame_fraction ≥ 0.15 or static_occupancy ≥ 0.85 within 30-second windows.
- high_variance_frame_fraction values fall in [0, 1] and static_occupancy values fall in [0, 1] for every row.
- Every excluded row carries a non-empty reason indicating which threshold (≥15% high-variance or ≥85% static occupancy) triggered exclusion.
- Videos at the boundary (exactly 0.15 high-variance fraction or 0.85 static occupancy) are handled per the ≥ threshold convention and spot-checked against manual inspection of a sample of flagged videos.

## Limitations

- Thresholds (15% high-variance frames, 85% static occupancy) and the 30-second window are calibrated to ≤60-second YouTube short videos; they may not transfer to longer or differently sourced footage.
- The finding is stated as a pipeline rule; the paper text (methods section) names 'Motion dynamics analysis' only as a filter step without publishing the full implementation, so exact Farneback parameters are not specified.
- The public StreamingCoT repository provides dataset artifacts and toolkit links but no changelog, so implementation drift between the described rule and released code cannot be verified.

## Evidence
