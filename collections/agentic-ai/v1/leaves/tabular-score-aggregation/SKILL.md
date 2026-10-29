---
name: tabular-score-aggregation
description: Use when a curated video corpus must be quality-gated by aesthetic score
  — e.g., short (≤60 s) YouTube-sourced clips needing a 0–10 CLIP+MLP aesthetic score
  and a ≥7/10 keep/discard decision logged per video in a CSV.
when_to_use_negative:
- The corpus has already passed aesthetic screening and only needs downstream pipeline
  stages (hierarchical captioning, dynamic QA generation, multimodal CoT synthesis,
  iterative validation).
- The quality gate of interest is a different screening criterion — HD resolution
  (≥720p), motion dynamics analysis, social validation (>5,000 interactions), or lexical
  density constraints — rather than aesthetic score.
- You need dataset assembly or geographic balancing (stratified geographic sampling
  via the YouTube API) rather than per-video aesthetic scoring and tabular keep/discard
  logging.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OpenCV
  role: Decodes each candidate video into representative frames that are fed to the
    aesthetic scorer
- name: CLIP (CLIP+MLP Aesthetic Score Predictor, following Open-Sora)
  role: Backbone of the 0–10 aesthetic score predictor applied to sampled frames;
    scores are thresholded at ≥7/10
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:5783b0f2e510d43210028d25ae37df0a9e94da18052b272610810a796f8b58a3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CLIP (CLIP+MLP Aesthetic Score Predictor, following Open-Sora)
  - CLIP (aesthetic score predictor backbone, CLIP+MLP as in Open-Sora)
  - OpenCV
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tabular-score-aggregation

## Summary

Applies the StreamingCoT aesthetic-quality gate: each candidate short video is scored with a CLIP+MLP Aesthetic Score Predictor (Open-Sora style) on a 0–10 scale, and per-video scores plus keep/discard decisions are aggregated into a tabular CSV, retaining only videos scoring ≥7/10 to maintain the aesthetic integrity of the dataset.

## When to use

Use when a curated video corpus must be quality-gated by aesthetic score — e.g., short (≤60 s) YouTube-sourced clips needing a 0–10 CLIP+MLP aesthetic score and a ≥7/10 keep/discard decision logged per video in a CSV.

## When NOT to use

- The corpus has already passed aesthetic screening and only needs downstream pipeline stages (hierarchical captioning, dynamic QA generation, multimodal CoT synthesis, iterative validation).
- The quality gate of interest is a different screening criterion — HD resolution (≥720p), motion dynamics analysis, social validation (>5,000 interactions), or lexical density constraints — rather than aesthetic score.
- You need dataset assembly or geographic balancing (stratified geographic sampling via the YouTube API) rather than per-video aesthetic scoring and tabular keep/discard logging.

## Inputs

- Curated short-video corpus (e.g., 5,745 YouTube-sourced videos, ≤60 seconds)
- Representative frames decoded per video (via OpenCV)

## Outputs

- aesthetic_scores.csv — one row per input video with video ID, aesthetic score (0–10), and binary keep/discard flag
- Filtered video subset satisfying the ≥7/10 aesthetic threshold

## How to apply

Load the curated video corpus (5,745 short videos, ≤60 seconds, YouTube-sourced) and decode each video into representative frames using OpenCV. Run the CLIP+MLP Aesthetic Score Predictor (the Open-Sora aesthetic predictor) over the sampled frames of each video to obtain an aesthetic score on the 0–10 scale. Apply the paper's aesthetic-filter threshold of ≥7/10 as the decision point: keep videos meeting or exceeding it, discard the rest. Write aesthetic_scores.csv listing each video ID, its aesthetic score, and a binary keep/discard flag. Validate that the CSV has one row per input video, every retained video has a score in 7.0–10.0, and all scores fall within 0–10. This gate runs as part of the paper's multimodal quality screening, alongside social validation (>5,000 interactions), lexical density constraints, HD resolution (≥720p), and motion dynamics analysis.

## Related tools

- **OpenCV** (Decodes each candidate video into representative frames that are fed to the aesthetic scorer)
- **CLIP (CLIP+MLP Aesthetic Score Predictor, following Open-Sora)** (Backbone of the 0–10 aesthetic score predictor applied to sampled frames; scores are thresholded at ≥7/10)

## Examples

```
python aesthetic_filter.py --corpus curated_videos/ --decoder opencv --predictor clip_mlp_opensora --threshold 7.0 --out aesthetic_scores.csv
```

## Evaluation signals

- aesthetic_scores.csv exists and contains exactly one row per input video with fields for video ID and score
- Every retained video has a numeric score in the range 7.0–10.0
- All scores in the CSV fall within 0–10 (no out-of-range or missing values)
- The keep/discard flag is binary and consistent with the threshold: keep iff score ≥ 7.0
- The number of discarded rows equals the number of videos with score < 7.0 in the input corpus

## Limitations

- The ≥7/10 threshold is fixed by the paper's filter; no per-category or adaptive recalibration is described.
- The score depends on which frames are sampled from each video; the source material does not specify a sampling rate or frame count.
- Videos are sourced from YouTube and remain subject to original content creators' rights; the dataset is released for non-commercial research and users must comply with YouTube's Terms of Service.

## Evidence
