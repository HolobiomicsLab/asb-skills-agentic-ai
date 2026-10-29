---
name: video-quality-screening-pipeline
description: Use when building a curated video corpus for VideoQA training and you
  must screen candidates by measured pixel resolution and aesthetic score before annotation.
when_to_use_negative:
- Input videos are not short streaming clips (≤60 s) from a YouTube-sourced VideoQA
  corpus — the ≥720p and ≥7/10 thresholds were calibrated for this corpus
- You need semantic or content-based filtering (e.g., social validation >5,000 interactions,
  lexical density, motion dynamics analysis) rather than pixel-resolution and aesthetic-score
  screening
- Videos are already annotated QA assets in the StreamingCoT dataset (bbox/, final_cot/,
  key_frames/ directories) and no re-screening is intended
edam_operation: http://edamontology.org/operation_0370
tools:
- name: FFmpeg
  role: FFprobe (FFmpeg) extracts each video stream's encoded height/width in pixels
    to enforce the ≥720p HD resolution threshold
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: OpenCV
  role: Decodes each candidate video into representative frames that are sampled for
    aesthetic scoring
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: CLIP (CLIP+MLP Aesthetic Score Predictor, as in Open-Sora)
  role: Backbone of the aesthetic scorer producing a 0–10 score used to apply the
    ≥7/10 keep threshold
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:6b917ed05beb297dbc3193bed52aac1155e42d91aa5881220c0fe78cc6f0561f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - CLIP (CLIP+MLP Aesthetic Score Predictor, as in Open-Sora)
  - FFmpeg
  - OpenCV
  edam_operation: http://edamontology.org/operation_0370
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# video-quality-screening-pipeline

## Summary

A multimodal quality-screening stage of the StreamingCoT construction pipeline that filters curated short videos (≤60 s, sourced via YouTube API) on resolution and aesthetics: FFmpeg/FFprobe enforces the HD resolution threshold (≥720p) and the CLIP+MLP Aesthetic Score Predictor (Open-Sora) enforces aesthetic scoring ≥7/10.

## When to use

Use when building a curated video corpus for VideoQA training and you must screen candidates by measured pixel resolution and aesthetic score before annotation.

## When NOT to use

- Input videos are not short streaming clips (≤60 s) from a YouTube-sourced VideoQA corpus — the ≥720p and ≥7/10 thresholds were calibrated for this corpus
- You need semantic or content-based filtering (e.g., social validation >5,000 interactions, lexical density, motion dynamics analysis) rather than pixel-resolution and aesthetic-score screening
- Videos are already annotated QA assets in the StreamingCoT dataset (bbox/, final_cot/, key_frames/ directories) and no re-screening is intended

## Inputs

- Curated short-video corpus (5,745 videos, each ≤60 seconds) collected via YouTube API
- Per-video video files passed to FFprobe/FFmpeg for stream metadata extraction
- Decoded representative frames per video (via OpenCV) fed to the CLIP+MLP aesthetic predictor

## Outputs

- resolution_screening.csv — one row per probed video with measured width/height (pixels) and a keep/drop decision
- aesthetic_scores.csv — one row per input video with video ID, aesthetic score (0–10), and binary keep/discard flag
- Filtered HD, high-aesthetic video subset feeding downstream hierarchical captioning and QA generation

## How to apply

Run FFprobe (part of FFmpeg) on every file in the curated video corpus to extract each video stream's encoded height and width in pixels, and keep a video only when its height is ≥ 720 pixels, per the paper's 'HD resolution (≥720p)' criterion. Then decode each surviving video into representative frames with OpenCV and score the sampled frames with the CLIP+MLP Aesthetic Score Predictor (following Open-Sora) on a 0–10 scale. Retain videos whose aesthetic score meets or exceeds the paper's threshold of ≥7/10; low-resolution/heavily-compressed videos and videos scoring below 7/10 are discarded to preserve aesthetic integrity. Record per-video measured dimensions plus keep/drop decisions in resolution_screening.csv and video ID, aesthetic score, and binary keep/discard flag in aesthetic_scores.csv. This screening sits between geographic balancing and downstream hierarchical captioning in the pipeline (YouTube API → Geographic balancing → Multimodal quality screening), alongside sibling filters such as social validation (>5,000 interactions) and lexical density constraints.

## Related tools

- **FFmpeg** (FFprobe (FFmpeg) extracts each video stream's encoded height/width in pixels to enforce the ≥720p HD resolution threshold) — https://github.com/Fleeting-hyh/StreamingCoT
- **OpenCV** (Decodes each candidate video into representative frames that are sampled for aesthetic scoring) — https://github.com/Fleeting-hyh/StreamingCoT
- **CLIP (CLIP+MLP Aesthetic Score Predictor, as in Open-Sora)** (Backbone of the aesthetic scorer producing a 0–10 score used to apply the ≥7/10 keep threshold) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 VIDEO_ID.mp4
```

## Evaluation signals

- resolution_screening.csv exists and contains exactly one row per probed video
- Every retained (keep) row in resolution_screening.csv reports height ≥ 720 pixels, matching the paper's 'HD resolution (≥720p)' criterion
- aesthetic_scores.csv exists and contains one row per input video with fields for video ID and score
- Every retained video in aesthetic_scores.csv has a score in 7.0–10.0, and all scores fall within the 0–10 range
- Videos removed are low-resolution or heavily compressed, consistent with the paper's stated purpose of the resolution-screening step

## Limitations

- The 720p threshold removes all low-resolution or heavily compressed videos, which may discard otherwise usable content
- The ≥7/10 aesthetic cutoff relies on the CLIP+MLP (Open-Sora) predictor and can reject stylistically unusual but valid videos
- All videos are sourced from YouTube and remain subject to original content creators' rights; users must comply with YouTube's Terms of Service, so the corpus is restricted to non-commercial research use
- Screening applies only to the curated 5,745-video short-form (≤60 s) corpus; no thresholds are defined for longer-form or non-YouTube sources

## Evidence
