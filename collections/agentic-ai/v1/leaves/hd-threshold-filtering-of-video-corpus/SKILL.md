---
name: hd-threshold-filtering-of-video-corpus
description: Use when a video corpus needs resolution-based screening — e.g. deciding
  which short YouTube clips survive a 'Multimodal quality screening' gate before annotation.
when_to_use_negative:
- Input videos are already known to be HD and only need aesthetic or motion screening
- The goal is content-level quality assessment (e.g. aesthetic scoring ≥7/10) rather
  than resolution gating
- Working with static images or frame-only inputs, not encoded video files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3393
tools:
- name: FFmpeg
  role: FFprobe reads each video's encoded resolution (height/width in pixels) to
    evaluate the ≥720p HD gate
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:09e1738336e7b95418bbf1e8081f721550da9fe6dd45f303c84c6fef18be4394
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - FFmpeg
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hd-threshold-filtering-of-video-corpus

## Summary

A screening step from StreamingCoT's multimodal quality pipeline that uses FFmpeg to probe each curated video's encoded pixel dimensions and keep only HD content (height ≥ 720 px), dropping low-resolution or heavily compressed files.

## When to use

Use when a video corpus needs resolution-based screening — e.g. deciding which short YouTube clips survive a 'Multimodal quality screening' gate before annotation.

## When NOT to use

- Input videos are already known to be HD and only need aesthetic or motion screening
- The goal is content-level quality assessment (e.g. aesthetic scoring ≥7/10) rather than resolution gating
- Working with static images or frame-only inputs, not encoded video files

## Inputs

- curated short-video corpus (YouTube clips, ≤60 seconds)
- per-video media files readable by FFmpeg/FFprobe

## Outputs

- resolution_screening.csv — one row per video with measured width/height and a keep/drop decision
- subset of videos passing the HD (≥720p) threshold

## How to apply

Iterate over the curated video corpus (≤60 s YouTube clips) and run FFprobe from FFmpeg on each file to read the video stream's encoded height and width in pixels. Apply the StreamingCoT HD criterion, retaining a video only when its height is ≥ 720 px, and record the measured dimensions plus a keep/drop decision per video. Combine the per-video records into a single table and write it as resolution_screening.csv with one row per probed video. The threshold mirrors the paper's 'HD resolution (≥720p)' filter, applied alongside social validation (>5,000 interactions), lexical density constraints, motion dynamics analysis, and aesthetic scoring (≥7/10) within the 'YouTube API → Geographic balancing → Multimodal quality screening' stage.

## Related tools

- **FFmpeg** (FFprobe reads each video's encoded resolution (height/width in pixels) to evaluate the ≥720p HD gate) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 input.mp4 | awk -F',' '$2>=720 {print "keep"} $2<720 {print "drop"}'
```

## Evaluation signals

- resolution_screening.csv exists and contains exactly one row per probed video
- Every 'keep' row reports a video height ≥ 720 px, matching the paper's 'HD resolution (≥720p)' criterion
- Each row carries both measured width and height plus an explicit keep/drop decision
- Retained set excludes any video flagged as low-resolution or heavily compressed

## Limitations

- The threshold assumes the FFprobe-reported encoded height is a faithful proxy for delivered visual quality; heavily compressed but nominally HD sources may still pass unless cross-checked by other filters
- The pipeline does not report a changelog or availability/license statement, so exact filtering script versions may not be recoverable from the repository

## Evidence

- [methods] The resolution screening step uses FFmpeg to preserve only HD content (videos at or above 720p resolution) and removes low-resolution or heavily compressed videos.: [evidence span withheld — anchor status: not_found]
- [methods] YouTube API → Geographic balancing → Multimodal quality screening: "YouTube API → Geographic balancing → Multimodal quality screening"
- [readme] Rigorous multimodal filtering: … HD resolution (≥720p) …: "Rigorous multimodal filtering:
  - Social validation (>5,000 interactions)
  - Lexical density constraints
  - HD resolution (≥720p)
  - Motion dynamics analysis
  - Aesthetic scoring (≥7/10)"
