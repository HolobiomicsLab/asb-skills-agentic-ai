---
name: optical-flow-motion-variance-analysis
description: Use when building a curated short-video corpus (≤60 s YouTube-sourced
  clips) and you need to screen candidate videos for motion blur or static segments
  as part of multimodal quality filtering before temporal annotation.
when_to_use_negative:
- The input is not video (e.g., already-extracted captions, QA pairs, or metadata
  tables) — the other StreamingCoT quality filters (lexical density constraints, aesthetic
  scoring ≥7/10, social validation >5,000 interactions) apply instead.
- The videos are long-form streams where per-window exclusion of whole files is inappropriate;
  this filter was designed to exclude entire ≤60 s candidate clips, not to trim segments
  within longer recordings.
- The corpus explicitly requires static or slow footage (e.g., time-lapse or slideshow
  benchmarks); this filter would systematically exclude such content by design.
tools:
- name: OpenCV
  role: Provides VideoCapture for frame loading and dense optical-flow computation
    (e.g., Farneback) used to derive per-frame flow-variance scores
- name: FFmpeg
  role: FFmpeg-backed codecs used by OpenCV to decode the YouTube-sourced video frames
    prior to flow analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:d12db1230b073b6dd3c4b1c076e554f2dbc050454d5a72e4d5849e383768c218
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FFmpeg
  - OpenCV
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# optical-flow-motion-variance-analysis

## Summary

A motion-dynamics screening filter from the StreamingCoT construction pipeline that computes dense frame-to-frame optical flow variance over 30-second windows to exclude videos with excessive motion blur (≥15% high-variance frames) or near-static content (≥85% static occupancy) before dataset annotation.

## When to use

Use when building a curated short-video corpus (≤60 s YouTube-sourced clips) and you need to screen candidate videos for motion blur or static segments as part of multimodal quality filtering before temporal annotation.

## When NOT to use

- The input is not video (e.g., already-extracted captions, QA pairs, or metadata tables) — the other StreamingCoT quality filters (lexical density constraints, aesthetic scoring ≥7/10, social validation >5,000 interactions) apply instead.
- The videos are long-form streams where per-window exclusion of whole files is inappropriate; this filter was designed to exclude entire ≤60 s candidate clips, not to trim segments within longer recordings.
- The corpus explicitly requires static or slow footage (e.g., time-lapse or slideshow benchmarks); this filter would systematically exclude such content by design.

## Inputs

- Sampled candidate videos (≤60 seconds, YouTube-sourced, e.g., MP4/WebM files decoded via FFmpeg-backed codecs)
- 30-second analysis windows over each video's frame sequence

## Outputs

- motion_filter_report.csv with one row per processed video containing fields: video_id, high_variance_frame_fraction, static_occupancy, excluded, reason
- Per-video motion statistics and exclusion decisions feeding the downstream multimodal quality screening stage of the StreamingCoT pipeline

## How to apply

Load each sampled candidate video (≤60 s, YouTube-sourced) with OpenCV's VideoCapture, which decodes frames through FFmpeg-backed codecs. Compute dense frame-to-frame optical flow (e.g., Farneback via OpenCV) and derive a per-frame flow-variance score quantifying motion dynamics. Within 30-second windows, mark frames as high-variance (excessive motion blur) or as belonging to static segments, then compute the fraction of high-variance frames and the temporal occupancy of static segments per video. Flag a video for exclusion if ≥15% of its frames are high-variance or if static segments occupy ≥85% of the window, recording the flag reason per video. The rationale is that StreamingCoT's downstream hierarchical captioning and QA generation require videos with meaningful temporal dynamics, so both extremes — blur-heavy and near-static footage — are discarded at the filtering stage alongside social validation, HD resolution (≥720p), lexical density, and aesthetic scoring (≥7/10) filters.

## Related tools

- **OpenCV** (Provides VideoCapture for frame loading and dense optical-flow computation (e.g., Farneback) used to derive per-frame flow-variance scores)
- **FFmpeg** (FFmpeg-backed codecs used by OpenCV to decode the YouTube-sourced video frames prior to flow analysis)

## Examples

```
import cv2; flow = cv2.calcOpticalFlowFarneback(prev_gray, gray, None, 0.5, 3, 15, 3, 5, 1.2, 0); frame_var = flow.var()
```

## Evaluation signals

- motion_filter_report.csv exists and contains exactly one row per processed video with the required fields (video_id, high_variance_frame_fraction, static_occupancy, excluded, reason).
- Every excluded=True row satisfies the thresholds: high_variance_frame_fraction ≥ 0.15 OR static_occupancy ≥ 0.85 within a 30-second window; excluded=False rows satisfy neither.
- All fraction/occupancy values fall in [0, 1]; static_occupancy is computed over 30-second windows rather than the whole file inconsistently.
- The reason field is non-empty and consistent with the corresponding exclusion flag (blur-driven vs. static-segment-driven).
- Videos surviving the filter should plausibly pass the companion quality gates (HD ≥720p, aesthetic ≥7/10), i.e., the excluded set overlaps only partially with other filters' exclusions.

## Limitations

- The 15% high-variance and 85% static-occupancy thresholds and the 30-second window are calibrated for StreamingCoT's ≤60 s short-video corpus; they may mislabel slow-motion, camera-pan, or blur-heavy but informative content if reused on other corpora.
- Dense optical flow (e.g., Farneback) is computationally expensive per frame, and motion blur can inflate flow variance, conflating capture artifacts with genuine dynamics.
- The article's provided sections name 'Motion dynamics analysis' only as a filter step without publishing exact Farneback parameters or code paths, so exact implementation details must be recovered from the toolkit repository.

## Evidence
