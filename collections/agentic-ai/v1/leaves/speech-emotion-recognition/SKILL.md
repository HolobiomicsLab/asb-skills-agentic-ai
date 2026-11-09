---
name: speech-emotion-recognition
description: Use when the input is a speech recording whose emotional tone (e.g. angry)
  must be identified before further editing or regeneration, as in AudioFab's emotion-reversed
  speech resynthesis use case.
when_to_use_negative:
- Input is music or non-speech audio — AudioFab exposes Music Emotion Recognition
  separately from Speech Emotion Recognition in the FunTTS MCP Servers.
- Only transcription is needed and no emotion analysis or regeneration is requested
  — Whisper alone suffices without invoking the Emotion Tool.
- Local models requiring FunTTS MCP Server deployment are unavailable and no API token
  is configured — the README instructs removing FunTTS_mcp_servers registration, so
  SER cannot run.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the task and schedules the Emotion
    Tool within the speech perception toolchain.
  repo: https://github.com/SmileHnu/AudioFab
- name: Emotion Tool (Speech Emotion Recognition)
  role: Performs the emotion classification on the input speech clip; listed under
    Speech Understanding in the FunTTS MCP Servers.
  repo: https://github.com/SmileHnu/AudioFab
- name: Whisper
  role: Transcribes the speech to text after emotion analysis so emotion-bearing words
    can be edited.
- name: Editing Tool
  role: Edits the transcript at the word level, replacing emotion-bearing words with
    opposite-emotion counterparts.
- name: CosyVoice2
  role: Resynthesizes the edited transcript with the target emotional tone, conditioning
    on the original speaker to preserve timbre.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:f1ca8ded9a5709aaf8283d09606f6fa6bb1621c74be03601f4e7d2aaaef8a25c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - CosyVoice2
  - Editing Tool
  - Emotion Tool (Speech Emotion Recognition)
  - Whisper
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# speech-emotion-recognition

## Summary

Classify the emotional tone of a speech recording as the first stage of AudioFab's speech perception workflow (Figure 2, Case 2), so that a transcript can be rewritten and resynthesized with the opposite emotion while preserving speaker timbre.

## When to use

Use when the input is a speech recording whose emotional tone (e.g. angry) must be identified before further editing or regeneration, as in AudioFab's emotion-reversed speech resynthesis use case.

## When NOT to use

- Input is music or non-speech audio — AudioFab exposes Music Emotion Recognition separately from Speech Emotion Recognition in the FunTTS MCP Servers.
- Only transcription is needed and no emotion analysis or regeneration is requested — Whisper alone suffices without invoking the Emotion Tool.
- Local models requiring FunTTS MCP Server deployment are unavailable and no API token is configured — the README instructs removing FunTTS_mcp_servers registration, so SER cannot run.

## Inputs

- Speech recording file (e.g. WAV audio with an emotional tone such as angry speech)
- Natural-language user command describing the emotion analysis / emotion-reversal goal (handled by the AudioFab LLM Planner)

## Outputs

- Emotion label with confidence score for the input clip (e.g. 'happy' at 91%)
- Subtask result consumed by the LLM Planner to guide subsequent Whisper transcription, Editing Tool edits, and CosyVoice2 resynthesis (final chain output: opposite_emotion_speech.wav)

## How to apply

Load the input speech clip (e.g. an angry-toned recording from the paper's video demo) and run the Emotion Tool, AudioFab's Speech Emotion Recognition service exposed via the FunTTS MCP Servers. The tool returns a style label with a confidence value (the paper reports e.g. 'happy' at 91% confidence), which the LLM Planner uses as a subtask result. This emotion analysis runs first in a four-tool chain: Emotion Tool → Whisper transcription → Editing Tool word-level edit → CosyVoice2 resynthesis. Rationale: AudioFab's LLM Planner decomposes the user command into subtasks, performs tool selection via MCP Server intelligent filtering, and invokes the emotion tool before downstream editing so the opposite-emotion target is known; the final CosyVoice2 step conditions on the original speaker to preserve timbre.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the task and schedules the Emotion Tool within the speech perception toolchain.) — https://github.com/SmileHnu/AudioFab
- **Emotion Tool (Speech Emotion Recognition)** (Performs the emotion classification on the input speech clip; listed under Speech Understanding in the FunTTS MCP Servers.) — https://github.com/SmileHnu/AudioFab
- **Whisper** (Transcribes the speech to text after emotion analysis so emotion-bearing words can be edited.)
- **Editing Tool** (Edits the transcript at the word level, replacing emotion-bearing words with opposite-emotion counterparts.)
- **CosyVoice2** (Resynthesizes the edited transcript with the target emotional tone, conditioning on the original speaker to preserve timbre.)

## Examples

```
conda activate AudioFab; python scripts/start_all.py  # then issue: 'Analyze the emotion of this speech recording and regenerate it with the opposite emotion while keeping the speaker's voice.'
```

## Evaluation signals

- The Emotion Tool returns a style label with a plausible confidence percentage (the paper's demo reports 'happy' at 91% confidence).
- All subtasks in the AudioFab planner's toolchain (emotion analysis, transcription, editing, resynthesis) report successful execution before the response is generated.
- The resynthesized clip (opposite_emotion_speech.wav) exhibits the reversed emotion while speaker timbre is preserved via CosyVoice2 speaker conditioning.
- The transcript from Whisper contains the original emotion-bearing words that the Editing Tool successfully rewrites to their opposite-emotion variants.

## Limitations

- Each integrated tool runs in its own MCP-based dependency environment; local deployment of FunTTS models consumes substantial computing resources, so some tools may need to be used via API (MD_TOKEN / HF_TOKEN).
- If local deployment is incomplete, the FunTTS services must be de-registered and Speech Emotion Recognition will be unavailable.
- The official paper link is pending the revised version, and the repository has no changelog, limiting traceability of tool-version-specific behavior.

## Evidence

- [results] In the speech perception case, AudioFab chains four tools: Emotion Tool analyzes the emotion in the audio, Whisper transcribes the audio, Editing Tool edits the audio at the word level, and Cosy Voice2 synthesizes speech with the emotional tone.: [evidence span withheld — anchor status: not_found]
