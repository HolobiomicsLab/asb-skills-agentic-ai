---
name: threshold-based-video-filtering
description: Use when building a curated video corpus for VideoQA/dataset construction
  and you need an objective quality gate to discard low-aesthetic-appeal videos before
  annotation or QA generation.
when_to_use_negative:
- The video collection already satisfies the other multimodal quality screens (social
  validation >5,000 interactions, HD ≥720p) and aesthetic filtering is not part of
  your curation criteria.
- Input is a single video for quality inspection rather than a corpus requiring systematic,
  threshold-based curation.
- You need semantic or content relevance filtering (e.g. lexical density constraints
  or motion dynamics analysis) rather than aesthetic-appeal scoring — those are separate
  filter steps in the StreamingCoT pipeline.
edam_operation: http://edamontology.org/operation_3926
edam_topics: []
tools:
- name: CLIP
  role: Backbone of the CLIP+MLP Aesthetic Score Predictor (as in Open-Sora) that
    produces the 0–10 aesthetic score per video
- name: OpenCV
  role: Used to decode videos into representative frames that are fed to the aesthetic
    score predictor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:d6cde5a1d822f1e79509cac63b67dbd304cf0cff6484631b66f7364ebad27cea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CLIP
  - OpenCV
  edam_operation: http://edamontology.org/operation_3926
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# threshold-based-video-filtering

## Summary

Implements the StreamingCoT aesthetic-quality gate: each candidate video is scored with a CLIP+MLP Aesthetic Score Predictor (following Open-Sora) and discarded if its 0–10 score falls below 7/10, preserving the aesthetic integrity of the curated 5,745-video corpus.

## When to use

Use when building a curated video corpus for VideoQA/dataset construction and you need an objective quality gate to discard low-aesthetic-appeal videos before annotation or QA generation.

## When NOT to use

- The video collection already satisfies the other multimodal quality screens (social validation >5,000 interactions, HD ≥720p) and aesthetic filtering is not part of your curation criteria.
- Input is a single video for quality inspection rather than a corpus requiring systematic, threshold-based curation.
- You need semantic or content relevance filtering (e.g. lexical density constraints or motion dynamics analysis) rather than aesthetic-appeal scoring — those are separate filter steps in the StreamingCoT pipeline.

## Inputs

- Curated video corpus of short videos (StreamingCoT: 5,745 videos, ≤60 seconds, sourced via YouTube API)
- Per-video representative/sampled frames (decoded with OpenCV)

## Outputs

- Per-video aesthetic scores on a 0–10 scale
- CSV file (e.g. aesthetic_scores.csv) with one row per video containing video ID, aesthetic score, and binary keep/discard flag
- Filtered subset of videos with score ≥7/10 retained for downstream annotation

## How to apply

Load the curated video corpus (5,745 short videos, each ≤60 seconds) and decode representative frames with OpenCV. Run the CLIP+MLP Aesthetic Score Predictor (the Open-Sora aesthetic predictor) over the sampled frames of each video to obtain a 0–10 aesthetic score. Apply the paper's hard threshold of ≥7/10, retaining only videos that meet or exceed it; this step is one node of the paper's multimodal quality screening that also includes social validation (>5,000 interactions), HD resolution (≥720p), lexical density constraints, and motion dynamics analysis. Emit a CSV listing each video ID, its aesthetic score, and a binary keep/discard flag so the gate is auditable and reproducible downstream.

## Related tools

- **CLIP** (Backbone of the CLIP+MLP Aesthetic Score Predictor (as in Open-Sora) that produces the 0–10 aesthetic score per video)
- **OpenCV** (Used to decode videos into representative frames that are fed to the aesthetic score predictor)

## Examples

```
python scripts/aesthetic_filter.py --corpus videos/ --predictor open_sora_clip_mlp --threshold 7.0 --output aesthetic_scores.csv
```

## Evaluation signals

- aesthetic_scores.csv exists and contains one row per input video with fields for video ID and score
- Every retained (keep=1) video has a score value in the range 7.0–10.0
- All scores in the output fall within the valid 0–10 range
- Each video in the corpus appears exactly once in the output CSV (one row per input video)
- The retained video set is consistent with the ≥7/10 threshold: no video below 7 is kept and no video at or above 7 is discarded

## Limitations

- The 7/10 cutoff is a corpus-design choice from the StreamingCoT paper; other curation goals may require a different threshold.
- Aesthetic scores measure visual appeal, not semantic relevance, so this gate alone is insufficient for corpus curation (the paper combines it with social validation, resolution, lexical density, and motion filters).
- Scores depend on the specific CLIP+MLP predictor (Open-Sora's); swapping in another predictor may shift the score distribution and require recalibration of the threshold.

## Evidence

- [other] The aesthetic-evaluation filter operates by scoring each candidate video with the CLIP+MLP Aesthetic Score Predictor (following Open-Sora) and discarding any video whose score falls below 7/10, in order to maintain the aesthetic integrity of the dataset.: [evidence span withheld — anchor status: not_found]
