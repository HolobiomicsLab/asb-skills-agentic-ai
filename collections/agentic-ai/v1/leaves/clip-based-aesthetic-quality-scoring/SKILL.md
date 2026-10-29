---
name: clip-based-aesthetic-quality-scoring
description: Use when a video collection pipeline needs an aesthetic-quality gate
  on decoded video frames — e.g., curating a short-video corpus where low-aesthetic-quality
  clips must be filtered out before annotation.
when_to_use_negative:
- The filtering goal is resolution, social popularity, lexical density, or motion
  dynamics rather than aesthetic quality — StreamingCoT handles these as separate
  screening criteria (e.g., HD ≥720p, >5,000 interactions).
- Input videos have already passed the aesthetic gate and the task is downstream annotation
  (per-second captioning, QA generation, CoT synthesis).
- The corpus contains long-form videos outside the ≤60-second short-video scope of
  the StreamingCoT pipeline.
tools:
- name: CLIP
  role: Backbone of the CLIP+MLP aesthetic score predictor (following Open-Sora) used
    to produce the 0–10 aesthetic score
- name: OpenCV
  role: Decodes each curated video into representative frames that are fed to the
    aesthetic predictor
- name: StreamingCoT construction toolkit
  role: Repository containing the dataset construction pipeline in which the aesthetic-scoring
    filter step operates
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:d4672c25d8ad23f75d67ecbd1e367639805fb61c6c67bd686110edb5800ad156
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - CLIP
  - CLIP (aesthetic score predictor backbone, CLIP+MLP as in Open-Sora)
  - OpenCV
  - StreamingCoT construction toolkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# clip-based-aesthetic-quality-scoring

## Summary

Scores each candidate video's visual aesthetic quality with the CLIP+MLP Aesthetic Score Predictor (as used in Open-Sora) on a 0–10 scale and discards videos below the 7/10 cutoff, as part of StreamingCoT's multimodal quality screening of its 5,745-video YouTube corpus.

## When to use

Use when a video collection pipeline needs an aesthetic-quality gate on decoded video frames — e.g., curating a short-video corpus where low-aesthetic-quality clips must be filtered out before annotation.

## When NOT to use

- The filtering goal is resolution, social popularity, lexical density, or motion dynamics rather than aesthetic quality — StreamingCoT handles these as separate screening criteria (e.g., HD ≥720p, >5,000 interactions).
- Input videos have already passed the aesthetic gate and the task is downstream annotation (per-second captioning, QA generation, CoT synthesis).
- The corpus contains long-form videos outside the ≤60-second short-video scope of the StreamingCoT pipeline.

## Inputs

- Decoded video frames from the curated short-video corpus (≤60 s YouTube videos, sampled via OpenCV)
- Video ID list for the candidate corpus

## Outputs

- Per-video aesthetic score on the 0–10 scale
- CSV (e.g., aesthetic_scores.csv) with one row per video: video ID, aesthetic score, and binary keep/discard flag

## How to apply

Decode each video from the curated corpus (StreamingCoT uses 5,745 short videos, ≤60 seconds) into representative frames with OpenCV. Run the CLIP+MLP Aesthetic Score Predictor — the Open-Sora aesthetic predictor, with CLIP as the scoring backbone — over sampled frames to obtain a 0–10 aesthetic score per video. Apply the paper's threshold of ≥7/10, retaining videos that meet or exceed it while the broader multimodal screening (social validation >5,000 interactions, HD ≥720p, motion dynamics analysis, lexical density constraints) runs alongside. Record per-video ID, score, and keep/discard decision in an output table (e.g., a CSV such as aesthetic_scores.csv). The rationale is to maintain the aesthetic integrity of the dataset prior to per-second captioning and reasoning-chain annotation.

## Related tools

- **CLIP** (Backbone of the CLIP+MLP aesthetic score predictor (following Open-Sora) used to produce the 0–10 aesthetic score)
- **OpenCV** (Decodes each curated video into representative frames that are fed to the aesthetic predictor)
- **StreamingCoT construction toolkit** (Repository containing the dataset construction pipeline in which the aesthetic-scoring filter step operates) — https://github.com/Fleeting-hyh/StreamingCoT

## Evaluation signals

- aesthetic_scores.csv exists and contains one row per input video with fields for video ID and score.
- Every retained (keep-flagged) video has an aesthetic score in the range 7.0–10.0, matching the ≥7/10 threshold.
- All scores, retained and discarded, fall within the valid 0–10 predictor scale.
- The count of kept videos is consistent with the other multimodal screening filters applied to the same corpus (social validation >5,000 interactions, HD ≥720p, motion dynamics, lexical density).

## Limitations

- The 7/10 cutoff is specific to StreamingCoT's curation goals; other corpora may require a different threshold.
- Aesthetic scoring is only one component of multimodal quality screening — a video passing the aesthetic gate can still be discarded by social validation, HD-resolution, lexical-density, or motion-dynamics filters.
- Scores depend on frame sampling; the article specifies decoding into representative frames but not a unique sampling strategy.

## Evidence

- [methods] Aesthetic scoring (≥7/10): "Aesthetic scoring (≥7/10)"
- [readme] Rigorous multimodal filtering incl. aesthetic scoring: "Rigorous multimodal filtering:
  - Social validation (>5,000 interactions)
  - Lexical density constraints
  - HD resolution (≥720p)
  - Motion dynamics analysis
  - Aesthetic scoring (≥7/10)"
- [readme] Corpus scope and screening pipeline: "5,745 high-quality short videos (≤60 seconds)"
