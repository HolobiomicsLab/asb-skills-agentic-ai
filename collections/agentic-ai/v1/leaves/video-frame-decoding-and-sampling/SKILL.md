---
name: video-frame-decoding-and-sampling
description: Use when a video curation pipeline must decode raw video files and produce
  per-frame samples as input to motion-dynamics screening (e.g., optical-flow variance
  over 30-second windows before dataset construction).
when_to_use_negative:
- The video corpus has already been motion-filtered and motion_filter_report.csv exists
  with exclusion decisions — re-decoding only to recompute identical statistics is
  wasteful
- The goal is content annotation (per-second captioning, QA generation, or CoT synthesis)
  rather than frame-level motion screening
- Input clips are longer than 60 s or not YouTube-sourced; the StreamingCoT filter's
  15%/85% thresholds and window logic were defined for ≤60 s candidate videos
tools:
- name: OpenCV
  role: VideoCapture for decoding frames and Farneback dense optical flow for per-frame
    variance computation
- name: FFmpeg
  role: Codec backend used by VideoCapture to convert/decode the YouTube-sourced video
    formats
- name: YouTube API
  role: Upstream source of the candidate videos fed into decoding and motion screening
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:ad5194a1a13603dcf39f2c44cbc95b6defcbd85a056da7f1a2c923ad4ad00689
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FFmpeg
  - OpenCV
  - YouTube API
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# video-frame-decoding-and-sampling

## Summary

Decodes short (≤60 s) YouTube-sourced candidate videos frame-by-frame with OpenCV's VideoCapture (FFmpeg-backed codecs) and samples frames within 30-second windows so dense optical-flow motion statistics can be computed in the StreamingCoT motion-dynamics filter.

## When to use

Use when a video curation pipeline must decode raw video files and produce per-frame samples as input to motion-dynamics screening (e.g., optical-flow variance over 30-second windows before dataset construction).

## When NOT to use

- The video corpus has already been motion-filtered and motion_filter_report.csv exists with exclusion decisions — re-decoding only to recompute identical statistics is wasteful
- The goal is content annotation (per-second captioning, QA generation, or CoT synthesis) rather than frame-level motion screening
- Input clips are longer than 60 s or not YouTube-sourced; the StreamingCoT filter's 15%/85% thresholds and window logic were defined for ≤60 s candidate videos

## Inputs

- Short (≤60 s) YouTube-sourced candidate video files that passed upstream screening (social validation >5,000 interactions, ≥720p HD resolution, aesthetic score ≥7/10)
- 30-second analysis window definition applied over the decoded frame stream

## Outputs

- Decoded per-frame image stream per video, grouped into 30-second windows
- Per-frame flow-variance scores from dense Farneback optical flow
- motion_filter_report.csv with columns: video_id, high_variance_frame_fraction, static_occupancy, excluded, reason

## How to apply

Load each sampled candidate video (≤60 s, YouTube-sourced) using OpenCV's VideoCapture, which decodes frames through FFmpeg-backed codecs. Read frames sequentially to obtain a uniform per-frame stream across the full clip, keeping frame order and timestamps so that frames can be grouped into 30-second windows. Feed consecutive decoded frame pairs into a dense frame-to-frame optical-flow computation (e.g., Farneback via OpenCV) to derive a per-frame flow-variance score quantifying motion dynamics. The rationale is that StreamingCoT's multimodal quality screening (YouTube API → Geographic balancing → Multimodal quality screening) requires per-frame motion statistics before exclusion decisions are made: a video is flagged if ≥15% of its frames are high-variance (motion blur) or if static segments occupy ≥85% of a 30-second window. Record per-video statistics to motion_filter_report.csv with the fields video_id, high_variance_frame_fraction, static_occupancy, excluded, reason.

## Related tools

- **OpenCV** (VideoCapture for decoding frames and Farneback dense optical flow for per-frame variance computation)
- **FFmpeg** (Codec backend used by VideoCapture to convert/decode the YouTube-sourced video formats)
- **YouTube API** (Upstream source of the candidate videos fed into decoding and motion screening)

## Examples

```
import cv2; cap = cv2.VideoCapture(video_path); ret, frame = cap.read(); prev = frame; ret, frame = cap.read(); flow = cv2.calcOpticalFlowFarneback(cv2.cvtColor(prev, cv2.COLOR_BGR2GRAY), cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY), None, 0.5, 3, 15, 3, 5, 1.2, 0)
```

## Evaluation signals

- motion_filter_report.csv exists and contains one row per processed video with the required fields (video_id, high_variance_frame_fraction, static_occupancy, excluded, reason)
- high_variance_frame_fraction and static_occupancy values lie in [0, 1] and are consistent with the number of decoded frames per 30-second window
- Each excluded flag is exactly consistent with the thresholds: excluded when ≥15% of frames are high-variance or static segments occupy ≥85% of the window, with a recorded flag reason
- Decoded frame counts match expected frame rates over the ≤60 s clip duration, and no video fails silently during VideoCapture decoding
- Videos filtered out here should not appear in downstream StreamingCoT annotation outputs (bbox/, final_cot/, key_frames/ directories)

## Limitations

- Screening is exclusionary: clips with ≥15% high-variance frames (motion blur) or ≥85% static occupancy in a 30-second window are removed, so this step only prunes the corpus and produces no annotated data
- All videos must remain subject to original content creators' rights and YouTube's Terms of Service, which constrains redistribution of decoded frames
- Decoding depends on FFmpeg codec support for the source formats; unsupported encodings will cause VideoCapture read failures

## Evidence

- [methods] Load the sampled candidate videos (≤60 s, YouTube-sourced) using OpenCV's VideoCapture, decoding frames with FFmpeg-backed codecs.: [evidence span withheld — anchor status: not_found]
- [results] The motion-dynamics filter computes optical flow variance to flag videos for exclusion when at least 15% of frames show high variance.: [evidence span withheld — anchor status: not_found]
