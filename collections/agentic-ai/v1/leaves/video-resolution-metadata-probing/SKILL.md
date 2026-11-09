---
name: video-resolution-metadata-probing
description: Use when a video corpus must be filtered by encoded resolution before
  downstream annotation, and per-video width/height metadata plus keep/drop decisions
  need to be recorded (e.g. enforcing an 'HD resolution (>=720p)' criterion).
when_to_use_negative:
- Videos are already known to meet the resolution floor (e.g. pre-filtered source
  streams) and only other screening criteria (social validation, aesthetic scoring,
  lexical density) remain
- The task requires content-based quality judgment (aesthetics, motion dynamics) rather
  than encoded stream metadata extraction
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3892
tools:
- name: FFmpeg
  role: Provides FFprobe, used to read each video's encoded stream width/height for
    the resolution keep/drop decision
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:067eddf175045cb17d9dda46355984177bf12078b8a650ae59a3b1fe2eb793f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - FFmpeg
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3892
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Video resolution metadata probing

## Summary

Probes each video's encoded stream dimensions with FFprobe (part of FFmpeg) and screens the corpus to preserve only HD content (height >= 720 pixels), as used in StreamingCoT's multimodal quality screening of its curated YouTube video corpus.

## When to use

Use when a video corpus must be filtered by encoded resolution before downstream annotation, and per-video width/height metadata plus keep/drop decisions need to be recorded (e.g. enforcing an 'HD resolution (>=720p)' criterion).

## When NOT to use

- Videos are already known to meet the resolution floor (e.g. pre-filtered source streams) and only other screening criteria (social validation, aesthetic scoring, lexical density) remain
- The task requires content-based quality judgment (aesthetics, motion dynamics) rather than encoded stream metadata extraction

## Inputs

- Curated short-video corpus (e.g. 5,745 <=60-second videos sourced from YouTube) as local video files
- HD resolution threshold from the paper's multimodal quality screening (>=720p, i.e. height >= 720 pixels)

## Outputs

- resolution_screening.csv with one row per probed video containing encoded width, encoded height, and a keep/drop decision
- Filtered subset of HD videos (height >= 720 pixels) retained for the downstream pipeline

## How to apply

Iterate over the curated video corpus and run FFprobe on each file to extract the video stream's encoded height and width in pixels. Apply the paper's HD threshold from the multimodal quality screening stage, keeping a video only when its height is >= 720 pixels (720p), and record the measured dimensions plus a keep/drop decision per video. Assemble the per-video measurements and decisions into a single table and write it as resolution_screening.csv with one row per video. Rationale: low-resolution or heavily compressed videos would degrade StreamingCoT's per-second dense captioning, keyframe grounding, and spatiotemporal object extraction, so resolution is screened alongside social validation (>5,000 interactions), lexical density constraints, motion dynamics analysis, and aesthetic scoring (>=7/10).

## Related tools

- **FFmpeg** (Provides FFprobe, used to read each video's encoded stream width/height for the resolution keep/drop decision) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 video.mp4
```

## Evaluation signals

- resolution_screening.csv exists and contains exactly one row per probed video
- Every row marked keep reports a height of at least 720 pixels, matching the paper's 'HD resolution (>=720p)' criterion
- Every dropped row reports height < 720 pixels or missing/unreadable stream dimensions
- The retained subset's size is consistent with the other multimodal filters (social validation >5,000 interactions, aesthetic score >=7/10) applied in the same stage
- Reported widths/heights are plausible positive pixel dimensions consistent with FFprobe output for each source file

## Limitations

- The screen uses encoded stream dimensions only; it does not detect upscale artifacts or effective (perceptual) resolution of heavily compressed videos
- The paper does not publish the exact FFprobe command or per-video format of resolution_screening.csv in the provided text, so exact output schema must be validated against the construction toolkit repository
- StreamingCoT videos are sourced from YouTube and subject to original content creators' rights and YouTube's Terms of Service, constraining redistribution of screened corpus files

## Evidence
