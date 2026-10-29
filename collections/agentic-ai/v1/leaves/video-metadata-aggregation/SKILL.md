---
name: video-metadata-aggregation
description: 'Use when filtering candidate YouTube videos by popularity: input is
  per-video metadata records with likes, comments, and shares fields, and you need
  to retain only socially validated videos above an interaction threshold.'
when_to_use_negative:
- The metadata records lack likes, comments, or shares fields, so an aggregated interaction
  count cannot be computed
- The goal is a quality filter on video content itself (HD resolution ≥720p, aesthetic
  score ≥7/10, motion dynamics, lexical density) rather than social popularity
- Candidate videos come from platforms other than YouTube, whose metadata fields differ
  from the YouTube API schema
tools:
- name: YouTube API
  role: Source of the per-video metadata records (likes, comments, shares) used in
    the Video Collection & Filtering step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:fda97bae0343d79e3f8634e9ebbc7e91eda8e95132761f9faf9860d5627fa068
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - YouTube API
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# video-metadata-aggregation

## Summary

Aggregates per-video social engagement metrics (likes, comments, shares) from YouTube API metadata into a single interaction_total score, then applies the StreamingCoT social-validation filter that retains only videos exceeding 5,000 aggregated interactions.

## When to use

Use when filtering candidate YouTube videos by popularity: input is per-video metadata records with likes, comments, and shares fields, and you need to retain only socially validated videos above an interaction threshold.

## When NOT to use

- The metadata records lack likes, comments, or shares fields, so an aggregated interaction count cannot be computed
- The goal is a quality filter on video content itself (HD resolution ≥720p, aesthetic score ≥7/10, motion dynamics, lexical density) rather than social popularity
- Candidate videos come from platforms other than YouTube, whose metadata fields differ from the YouTube API schema

## Inputs

- Sample of video metadata records (one record per YouTube video, sourced via the YouTube API), containing likes, comments, and shares fields

## Outputs

- social_validation_passed.csv: retained video records, including the computed interaction_total column, with every row >5,000 aggregated interactions

## How to apply

Load the per-video metadata records (one record per YouTube video, sourced via the YouTube API) into a pandas DataFrame. For each video, compute interaction_total as the sum of the likes, comments, and shares fields. Apply the social-validation filter keeping only rows where interaction_total is strictly greater than 5,000, and write the retained records (including the computed interaction_total column) to social_validation_passed.csv. This step is part of the StreamingCoT Video Collection & Filtering pipeline (YouTube API → Geographic balancing → Multimodal quality screening), where it serves as the social signal that a video is high-quality before downstream lexical-density, HD-resolution, motion-dynamics, and aesthetic filters are applied.

## Related tools

- **YouTube API** (Source of the per-video metadata records (likes, comments, shares) used in the Video Collection & Filtering step)

## Examples

```
df['interaction_total'] = df['likes'] + df['comments'] + df['shares']; df[df.interaction_total > 5000].to_csv('social_validation_passed.csv', index=False)
```

## Evaluation signals

- Every row in social_validation_passed.csv has interaction_total > 5,000
- Any input record with aggregated interactions at or below 5,000 is excluded from the output
- Each output row carries a computed interaction_total column equal to the sum of likes, comments, and shares for that video
- The set of retained videos is a strict subset of the input sample
- Retained videos can proceed to the remaining multimodal quality screening filters (HD ≥720p, aesthetic ≥7/10, motion dynamics, lexical density)

## Limitations

- Videos with exactly 5,000 interactions are excluded because the threshold is strictly greater than 5,000
- The filter captures social popularity only; it does not guarantee visual or semantic quality, which is assessed by the subsequent HD-resolution, motion-dynamics, aesthetic, and lexical-density filters
- The YouTube API metadata sample is subject to YouTube's Terms of Service and original content creators' rights, constraining redistribution of the filtered corpus

## Evidence

- [methods] social-validation filter aggregates per-video social interactions: [evidence span withheld — anchor status: not_found]
