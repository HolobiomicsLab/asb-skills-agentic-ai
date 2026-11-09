---
name: temporal-word-rate-threshold-filtering
description: Use when curating short video datasets (StreamingCoT videos are ≤60 seconds)
  and you must discard samples whose speech content is too dense, per the paper's
  lexical-density screening of ≤2 words per 10s interval.
when_to_use_negative:
- The dataset's filtering goals are other multimodal screens — social validation (>5,000
  interactions), HD resolution (≥720p), motion dynamics analysis, or aesthetic scoring
  (≥7/10) — which this speech-based filter does not address
- Videos are long-form (>60 seconds) such that the 10-second interval segmentation
  assumption (≤6 intervals per video) from the task card no longer holds without re-deriving
  thresholds
- The intended curation criterion is visual/temporal content (e.g. per-second dense
  captions or Dynamic Semantic Fusion segmentation), not spoken-word density
edam_operation: http://edamontology.org/operation_3695
tools:
- name: FFmpeg
  role: 'Extracts the audio track from each sample video before speech recognition
    (per the task card workflow: ''Extract the audio track from each sample video
    using FFmpeg'').'
- name: Whisper
  role: Automatic speech recognition producing the timestamped transcripts whose per-10-second
    word counts drive the lexical density decision (per the task card workflow step
    2).
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:614a5bfb9fc1613550bdbed908471cbf90d1a16c7007ecde6a6519d4bb7a50c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FFmpeg
  - Whisper
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Temporal word-rate threshold filtering

## Summary

Filters videos out of a multimodal dataset curation pipeline by transcribing their audio tracks with Whisper and discarding any video whose transcript exceeds two words in any 10-second interval. This enforces the 'lexical density constraints' used in the StreamingCoT multimodal quality screening so retained samples remain visually dominated rather than speech-dominated.

## When to use

Use when curating short video datasets (StreamingCoT videos are ≤60 seconds) and you must discard samples whose speech content is too dense, per the paper's lexical-density screening of ≤2 words per 10s interval.

## When NOT to use

- The dataset's filtering goals are other multimodal screens — social validation (>5,000 interactions), HD resolution (≥720p), motion dynamics analysis, or aesthetic scoring (≥7/10) — which this speech-based filter does not address
- Videos are long-form (>60 seconds) such that the 10-second interval segmentation assumption (≤6 intervals per video) from the task card no longer holds without re-deriving thresholds
- The intended curation criterion is visual/temporal content (e.g. per-second dense captions or Dynamic Semantic Fusion segmentation), not spoken-word density

## Inputs

- Short video files (≤60 seconds, e.g. YouTube-sourced MP4s from the StreamingCoT corpus)
- Audio tracks extracted from each video via FFmpeg
- Whisper timestamped transcripts (per-word timestamps on the video timeline)

## Outputs

- Per-video lexical density profile: word counts for each 10-second interval
- Keep/discard decision per video (discard if any 10s interval > 2 words)
- lexical_density_filter_results.csv containing per-video maximum interval word counts and decisions
- A retained video subset that passes the lexical density constraint for downstream multimodal quality screening

## How to apply

First extract each video's audio track with FFmpeg, then run Whisper automatic speech recognition on the audio to obtain timestamped transcripts. Segment each transcript into 10-second intervals aligned with the video timeline — since StreamingCoT videos are ≤60 seconds, each video yields at most 6 intervals. Compute the word count per 10-second interval to build each video's lexical density profile. Apply the decision rule from the task card: mark a video as discarded if ANY 10-second interval exceeds two words, otherwise retain it. Record all per-video maximum interval word counts and the keep/discard decisions to lexical_density_filter_results.csv so the filtering stage is auditable.

## Related tools

- **FFmpeg** (Extracts the audio track from each sample video before speech recognition (per the task card workflow: 'Extract the audio track from each sample video using FFmpeg').)
- **Whisper** (Automatic speech recognition producing the timestamped transcripts whose per-10-second word counts drive the lexical density decision (per the task card workflow step 2).)

## Examples

```
ffmpeg -i sample.mp4 -vn -acodec pcm_s16le audio.wav && whisper audio.wav --task transcribe --output_format   # then bin words into 10s intervals; discard if any interval > 2 words
```

## Evaluation signals

- lexical_density_filter_results.csv exists and contains a row for every candidate video with its maximum 10-second-interval word count and a keep/discard decision
- Every discarded video has at least one 10-second interval whose word count is strictly greater than 2; every retained video has all intervals ≤ 2 words
- No video has more than 6 intervals (consistent with the ≤60-second video length in the corpus)
- Word counts are computed from Whisper transcripts whose timestamps are aligned to the video timeline, so interval boundaries match the video's 0–60s clock
- The retained subset is consistent with the paper's reported multimodal filtering stack (lexical density applied alongside social, resolution, motion, and aesthetic filters)

## Limitations

- The two-words-per-10-seconds threshold is specific to StreamingCoT's curation goals; other datasets may need recalibrated thresholds
- Whisper ASR errors (misrecognitions, missing speech) directly change word counts and therefore keep/discard outcomes
- The paper's article text only names 'Lexical density constraints' as a filter step; the concrete ≤2 words/10s parameter and CSV output come from the source task card, so verify against the released construction toolkit

## Evidence
