---
name: social-interaction-threshold-filtering
description: Use when curating a YouTube-sourced video corpus and candidate metadata
  records carry likes, comments, and shares counts that must be screened by an aggregate
  popularity cutoff before further quality filtering.
when_to_use_negative:
- Metadata records lack likes, comments, or shares fields, or come from a platform
  that does not expose interaction counts — the aggregate score cannot be computed
- 'The screening target is visual or textual quality rather than audience engagement:
  HD resolution (≥720p), motion dynamics analysis, aesthetic scoring (≥7/10), and
  lexical density constraints are separate filter steps in the same pipeline'
- 'A non-strict cutoff is required: this skill excludes videos with exactly 5,000
  interactions because the threshold is strictly greater-than'
edam_operation: http://edamontology.org/operation_3695
tools:
- name: YouTube API
  role: Source of the per-video metadata records (likes, comments, shares) whose aggregated
    interaction total is screened against the 5,000 threshold
- name: pandas
  role: DataFrame engine used to load metadata, compute interaction_total = likes
    + comments + shares, and apply the strict > 5,000 filter before writing social_validation_passed.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:d7bf88247b12c4dd6e1b8f981e90ada115445fd26051e60e09ed02d75d67c861
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - YouTube API
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# social-interaction-threshold-filtering

## Summary

A social-validation screening step from the StreamingCoT 'Video Collection & Filtering' stage: per-video likes, comments, and shares from YouTube API metadata are summed, and only videos with strictly more than 5,000 total interactions are retained. It operationalizes aggregate audience engagement as the first multimodal quality-screening criterion before temporal annotation.

## When to use

Use when curating a YouTube-sourced video corpus and candidate metadata records carry likes, comments, and shares counts that must be screened by an aggregate popularity cutoff before further quality filtering.

## When NOT to use

- Metadata records lack likes, comments, or shares fields, or come from a platform that does not expose interaction counts — the aggregate score cannot be computed
- The screening target is visual or textual quality rather than audience engagement: HD resolution (≥720p), motion dynamics analysis, aesthetic scoring (≥7/10), and lexical density constraints are separate filter steps in the same pipeline
- A non-strict cutoff is required: this skill excludes videos with exactly 5,000 interactions because the threshold is strictly greater-than

## Inputs

- Per-video YouTube API metadata records (one record per candidate video), each with likes, comments, and shares fields

## Outputs

- social_validation_passed.csv — retained video records only, each with original metadata columns plus a computed interaction_total column where every value is strictly > 5,000

## How to apply

Load one metadata record per candidate YouTube video (sourced via the YouTube API) into a pandas DataFrame and compute interaction_total as the sum of the likes, comments, and shares fields. Retain a video only when interaction_total is strictly greater than 5,000 — records at exactly 5,000 are excluded. Write the surviving records, augmented with the computed interaction_total column, to social_validation_passed.csv. In the StreamingCoT pipeline this social-validation cutoff is the first listed multimodal quality-screening criterion applied after geographic balancing, alongside lexical density constraints, HD resolution (≥720p), motion dynamics analysis, and aesthetic scoring (≥7/10). The rationale is to use aggregate audience engagement as an objective proxy for content quality before investing in per-second captioning and reasoning-chain annotation.

## Related tools

- **YouTube API** (Source of the per-video metadata records (likes, comments, shares) whose aggregated interaction total is screened against the 5,000 threshold)
- **pandas** (DataFrame engine used to load metadata, compute interaction_total = likes + comments + shares, and apply the strict > 5,000 filter before writing social_validation_passed.csv)

## Examples

```
df['interaction_total'] = df['likes'] + df['comments'] + df['shares']; df[df['interaction_total'] > 5000].to_csv('social_validation_passed.csv', index=False)
```

## Evaluation signals

- Every row in social_validation_passed.csv has interaction_total > 5,000
- Any input record with likes + comments + shares ≤ 5,000 (including exactly 5,000) is absent from the output
- For each retained record, interaction_total equals the sum of its likes, comments, and shares fields
- The retained set is a strict subset of the input candidate records and preserves original metadata columns plus the interaction_total column
- After all multimodal filters in the pipeline, the corpus matches the README's curated scale of 5,745 short videos (≤60 seconds)

## Limitations

- The paper specifies only a raw aggregate cutoff (>5,000 interactions) with no normalization by video age or view count, so long-lived or heavily-viewed videos are inherently favored
- Records missing likes, comments, or shares fields cannot be scored, and boundary records at exactly 5,000 interactions are excluded by the strict inequality
- Passing social validation alone does not admit a video to the StreamingCoT dataset — lexical density, HD resolution (≥720p), motion dynamics, and aesthetic scoring (≥7/10) filters apply within the same multimodal quality-screening stage

## Evidence
