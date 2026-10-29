---
name: temporal-caption-segment-counting
description: Use when you hold a deposited video-annotation corpus (e.g., StreamingCoT's
  final_cot/ and initial_cot/ JSONL trees keyed by YouTube ID) and must reproduce
  reported per-second caption counts, DSF segment counts, video counts, or mean video
  duration.
when_to_use_negative:
- The goal is to construct new annotations (Hierarchical Captioning, QA generation,
  or CoT synthesis) rather than reproduce counts of an already-deposited dataset
- Input is raw video without per-second caption JSONL files — duration must then be
  measured with OpenCV/FFmpeg directly, not from caption coverage
- You need to evaluate annotation quality (temporal consistency, semantic completeness)
  rather than produce aggregate composition statistics
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: OpenCV
  role: Cross-check per-video durations (seconds) against the source videos, which
    are capped at 60 seconds, alongside caption-coverage-derived durations
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: Python json module
  role: Parse final_cot/ and initial_cot/ JSONL line by line and the sec_<s>_idx_<f>.json
    bounding-box files to tally captions, segments, and boxes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:191d88189db8f24e1d977e62c0432f5a4d8b2bf60a52e18645e78480590faac8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - OpenCV
  - Python json module
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# temporal-caption-segment-counting

## Summary

A counting/statistics-derivation skill for temporally annotated streaming-video datasets: it enumerates per-video annotation directories, parses per-second dense-caption JSONL files, and tallies Dynamic Semantic Fusion (DSF) segments to reproduce dataset composition statistics such as caption counts, segment counts, video counts, and mean duration.

## When to use

Use when you hold a deposited video-annotation corpus (e.g., StreamingCoT's final_cot/ and initial_cot/ JSONL trees keyed by YouTube ID) and must reproduce reported per-second caption counts, DSF segment counts, video counts, or mean video duration.

## When NOT to use

- The goal is to construct new annotations (Hierarchical Captioning, QA generation, or CoT synthesis) rather than reproduce counts of an already-deposited dataset
- Input is raw video without per-second caption JSONL files — duration must then be measured with OpenCV/FFmpeg directly, not from caption coverage
- You need to evaluate annotation quality (temporal consistency, semantic completeness) rather than produce aggregate composition statistics

## Inputs

- StreamingCoT repository/dataset directory from github.com/Fleeting-hyh/StreamingCoT with per-video subdirectories keyed by YouTube ID under bbox/, final_cot/, initial_cot/, key_frames/
- final_cot/VIDEO_ID.jsonl and initial_cot/VIDEO_ID.jsonl files (JSON Lines, one record per second/segment)
- Per-second bounding-box JSON files matching the pattern sec_<s>_idx_<f>.json (optional cross-check of per-second coverage)

## Outputs

- dataset_statistics.json containing one named field per statistic (video count, mean duration, dense subtitle count, DSF segment count, QA pair count, bounding-box count)
- Per-video duration estimates in seconds derived from caption coverage
- Tallies of per-second dense captions and DSF semantic segments per video

## How to apply

Enumerate the per-video subdirectories (keyed by YouTube ID) under bbox/, final_cot/, initial_cot/, and key_frames/ to obtain the video count. Parse the final_cot/ and initial_cot/ JSONL files line by line with Python's json module, counting per-second dense caption records (per-second captioning from Hierarchical Captioning) and the temporally-dependent semantic segments produced by similarity-based Dynamic Semantic Fusion (DSF). Derive per-video durations from the span of per-second caption coverage, cross-check with OpenCV against source videos (capped at 60 seconds), and compute the mean duration. Compile all statistics into a single JSON object with one named field per statistic (e.g., dataset_statistics.json) and accept the result when each count matches the reference values (5,000 videos, 25.6s mean, 243,185 dense subtitles, 68,940 segments) with zero or near-zero discrepancy.

## Related tools

- **OpenCV** (Cross-check per-video durations (seconds) against the source videos, which are capped at 60 seconds, alongside caption-coverage-derived durations) — https://github.com/Fleeting-hyh/StreamingCoT
- **Python json module** (Parse final_cot/ and initial_cot/ JSONL line by line and the sec_<s>_idx_<f>.json bounding-box files to tally captions, segments, and boxes)

## Examples

```
python -c "import json,glob; n=sum(1 for f in glob.glob('StreamingCoT/final_cot/*.jsonl') for _ in open(f)); print('dense_captions:',n)"
```

## Evaluation signals

- Each computed count matches the paper's Section 3.6 reference value within zero or near-zero discrepancy (5,000 videos; 25.6s mean duration; 243,185 dense subtitles; 68,940 segments; 34,470 QA pairs; 206,820 bounding boxes)
- Number of enumerated per-video subdirectories equals the number of distinct JSONL basenames in final_cot/ and initial_cot/
- Per-video caption coverage spans are contiguous per-second indices and every duration is ≤ 60 seconds
- Sum of DSF segments ≤ sum of per-second dense captions (fusion merges captions into longer temporally-dependent segments)
- Output is a single JSON object with one named field per statistic, written to dataset_statistics.json

## Limitations

- Reported composition (e.g., 5,000 videos) differs from the README's raw curated corpus of 5,745 high-quality short videos, so counts reflect the post-filtering/post-validation subset and may drift if the repository changes
- Duration inference from caption coverage assumes complete per-second captioning; gaps in coverage would bias the mean-duration estimate (mitigate via OpenCV cross-check)
- The anonymous access portal (https://anonymous.4open.science/) may not mirror the GitHub tree, so directory patterns like sec_<s>_idx_<f>.json may not resolve identically

## Evidence

- [full_text] workflow: enumerate dirs and parse JSONL for stats: [evidence span withheld — anchor status: not_found]
