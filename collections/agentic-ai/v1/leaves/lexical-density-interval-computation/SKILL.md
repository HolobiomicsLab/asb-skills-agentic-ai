---
name: lexical-density-interval-computation
description: 'Use when building the StreamingCoT multimodal quality screening stage:
  you have short (≤60 s) video samples with extracted audio and need to retain only
  videos with sparse speech, i.e. ≤2 transcript words per 10-second interval.'
when_to_use_negative:
- Videos longer than 60 seconds — the pipeline's assumption of at most 6 ten-second
  intervals per video no longer holds and the threshold was calibrated for StreamingCoT's
  short-video corpus
- You need a true linguistic lexical-density metric (content-word vs function-word
  ratio) — this skill uses raw transcript word counts per interval as the filtering
  proxy, not a lexical richness measure
- The corpus has already passed multimodal quality screening or contains no audio
  track, so there is no transcript to interval-segment
edam_operation: http://edamontology.org/operation_3695
tools:
- name: FFmpeg
  role: Extracts the audio track from each sample video so Whisper ASR can transcribe
    it
- name: Whisper
  role: Automatic speech recognition producing the timestamped transcripts whose 10-second
    interval word counts are the filter signal
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:176b3abbc795caacbfe0cf6a501b350c78c660b5aa6573d786dd9e9a19f0f7b9
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
yes_rate: 0.8
n_claims: 5
---

# lexical-density-interval-computation

## Summary

Computes a per-video lexical density profile — word counts within fixed 10-second transcript intervals from Whisper ASR output — to implement the StreamingCoT audio-content filter that discards videos whose any 10-second interval exceeds two words.

## When to use

Use when building the StreamingCoT multimodal quality screening stage: you have short (≤60 s) video samples with extracted audio and need to retain only videos with sparse speech, i.e. ≤2 transcript words per 10-second interval.

## When NOT to use

- Videos longer than 60 seconds — the pipeline's assumption of at most 6 ten-second intervals per video no longer holds and the threshold was calibrated for StreamingCoT's short-video corpus
- You need a true linguistic lexical-density metric (content-word vs function-word ratio) — this skill uses raw transcript word counts per interval as the filtering proxy, not a lexical richness measure
- The corpus has already passed multimodal quality screening or contains no audio track, so there is no transcript to interval-segment

## Inputs

- Sample videos (≤60 seconds, sourced from YouTube via YouTube API with geographic balancing)
- Audio tracks extracted from each video via FFmpeg
- Timestamped Whisper ASR transcripts per audio track

## Outputs

- Per-video lexical density profile (word count for each 10-second interval, ≤6 intervals per ≤60 s video)
- Keep/discard decision per video under the ≤2 words per 10 s constraint
- lexical_density_filter_results.csv with per-video maximum interval word counts and decisions

## How to apply

First extract each sample video's audio track with FFmpeg. Run Whisper automatic speech recognition on each audio track to obtain timestamped transcripts. Segment each transcript into 10-second intervals aligned with the video timeline — since StreamingCoT videos are ≤60 seconds, each video yields at most 6 intervals. Count words per interval to build the lexical density profile, then apply the constraint: discard the video if any 10-second interval exceeds two words, otherwise retain it. Finally write every video's maximum interval word count and its keep/discard decision to lexical_density_filter_results.csv so the filter step is auditable alongside the other screening criteria (social validation >5,000 interactions, HD ≥720p, motion dynamics, aesthetic ≥7/10).

## Related tools

- **FFmpeg** (Extracts the audio track from each sample video so Whisper ASR can transcribe it)
- **Whisper** (Automatic speech recognition producing the timestamped transcripts whose 10-second interval word counts are the filter signal)

## Examples

```
ffmpeg -i sample_video.mp4 -vn sample_audio.wav
```

## Evaluation signals

- lexical_density_filter_results.csv contains exactly one maximum-interval word count and one keep/discard decision per sample video
- No retained video has any 10-second interval with more than 2 words; every discarded video has at least one interval exceeding 2 words
- Number of intervals per video equals ceil(duration/10) and is at most 6 for the ≤60 s StreamingCoT corpus
- Interval boundaries align with the video timeline (10 s bins starting at t=0), cross-checkable against Whisper timestamps
- Spot-check a few Whisper transcripts against the corresponding FFmpeg-extracted audio to confirm recognition quality before trusting word counts

## Limitations

- The ≤2 words per 10 s threshold is calibrated for StreamingCoT's 5,745 short videos (≤60 s); transferring it to longer or denser-speech corpora may discard most samples
- Filter quality depends on Whisper ASR accuracy — transcription errors or missed speech directly distort interval word counts
- The metric is a word-count-per-interval proxy, not a content-vs-function-word lexical density measure, despite the 'lexical density' label
- The README lists the constraint among filter criteria but provides no implementation details or repository changelog, so exact tooling must be reconstructed

## Evidence

- [methods] Lexical density constraints listed as a multimodal filter step: "Lexical density constraints"
