---
name: video-quality-filter-rule-application
description: Use when a video corpus (≤60 s YouTube-sourced clips) must be screened
  during multimodal quality filtering and you need to exclude clips whose motion statistics
  violate the ≥15% high-variance or ≥85% static-occupancy rules.
when_to_use_negative:
- The input videos have already passed motion-dynamics screening and the goal is a
  later pipeline stage (e.g., per-second hierarchical captioning or Dynamic Semantic
  Fusion segmentation).
- The videos are not short-form (≤60 s) YouTube-sourced candidates and no 30-second
  windowing convention applies.
- The task is a different screening rule (HD resolution ≥720p, aesthetic scoring ≥7/10,
  social validation >5,000 interactions, lexical density) rather than motion dynamics.
tools:
- name: OpenCV
  role: Decodes frames via VideoCapture and computes dense Farneback optical flow
    used for per-frame flow-variance scoring
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: FFmpeg
  role: FFmpeg-backed codecs used by OpenCV to decode the video frames prior to flow
    computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:e75be1db01243b3693138beec1db85415e568f8ee3b5c5e70c939a6bbe558aeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - FFmpeg
  - OpenCV
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# video-quality-filter-rule-application

## Summary

Applies the StreamingCoT motion-dynamics filter, which uses dense optical flow (Farneback via OpenCV) to exclude videos with excessive motion blur or static content, using the 15% high-variance-frame and 85% static-occupancy thresholds over 30-second windows.

## When to use

Use when a video corpus (≤60 s YouTube-sourced clips) must be screened during multimodal quality filtering and you need to exclude clips whose motion statistics violate the ≥15% high-variance or ≥85% static-occupancy rules.

## When NOT to use

- The input videos have already passed motion-dynamics screening and the goal is a later pipeline stage (e.g., per-second hierarchical captioning or Dynamic Semantic Fusion segmentation).
- The videos are not short-form (≤60 s) YouTube-sourced candidates and no 30-second windowing convention applies.
- The task is a different screening rule (HD resolution ≥720p, aesthetic scoring ≥7/10, social validation >5,000 interactions, lexical density) rather than motion dynamics.

## Inputs

- Sampled candidate videos (≤60 s, YouTube-sourced) decodable by OpenCV VideoCapture/FFmpeg

## Outputs

- motion_filter_report.csv with columns: video_id, high_variance_frame_fraction, static_occupancy, excluded, reason

## How to apply

Load each sampled candidate video with OpenCV's VideoCapture (FFmpeg-backed decoding), then compute dense frame-to-frame Farneback optical flow and derive a per-frame flow-variance score. Within each 30-second window, mark frames as high-variance (excessive motion blur) or as belonging to static segments, and compute the high-variance frame fraction and the temporal occupancy of static segments. Flag a video for exclusion if ≥15% of its frames are high-variance or if static segments occupy ≥85% of the window, recording the flag reason per video. Write per-video statistics and exclusion decisions to motion_filter_report.csv (video_id, high_variance_frame_fraction, static_occupancy, excluded, reason). This step is one of five multimodal screening rules in the StreamingCoT Video Collection & Filtering stage, alongside social validation (>5,000 interactions), lexical density constraints, HD resolution (≥720p), and aesthetic scoring (≥7/10).

## Related tools

- **OpenCV** (Decodes frames via VideoCapture and computes dense Farneback optical flow used for per-frame flow-variance scoring) — https://github.com/Fleeting-hyh/StreamingCoT
- **FFmpeg** (FFmpeg-backed codecs used by OpenCV to decode the video frames prior to flow computation)

## Examples

```
python scripts/motion_filter.py --input candidates/ --window 30 --high-variance-thresh 0.15 --static-occupancy-thresh 0.85 --out motion_filter_report.csv
```

## Evaluation signals

- motion_filter_report.csv exists and contains one row per processed video
- Each row carries all required fields: video_id, high_variance_frame_fraction, static_occupancy, excluded, reason
- Excluded flags are consistent with the 15%/85% thresholds applied over 30-second windows (excluded=True iff high_variance_frame_fraction ≥ 0.15 OR static_occupancy ≥ 0.85)
- high_variance_frame_fraction values fall in [0, 1] and static_occupancy values fall in [0, 1]
- Every excluded=True row has a non-empty reason recorded per video

## Limitations

- The paper describes motion dynamics analysis only as one screening criterion; the exact flow-variance definition and window boundary handling are not fully specified in the article text.
- No changelog is available for the StreamingCoT repository, so implementation-level details of the filter may have changed without documentation.
- Videos sourced from YouTube remain subject to original content creators' rights and YouTube's Terms of Service, constraining redistribution of filtered inputs.

## Evidence

- [methods] The motion-dynamics filter computes optical flow variance to flag videos for exclusion when at least 15% of frames show high variance (motion blur) or static segments occupy at least 85% of a 30-second window.: [evidence span withheld — anchor status: not_found]
- [methods] Compute dense frame-to-frame optical flow (e.g., Farneback via OpenCV) and derive a per-frame flow-variance score quantifying motion dynamics for each video.: [evidence span withheld — anchor status: not_found]
- [readme] HD resolution (≥720p)
  - Motion dynamics analysis
  - Aesthetic scoring (≥7/10): "Motion dynamics analysis"
- [readme] 1. **Video Collection & Filtering**  
   YouTube API → Geographic balancing → Multimodal quality screening: "YouTube API → Geographic balancing → Multimodal quality screening"
