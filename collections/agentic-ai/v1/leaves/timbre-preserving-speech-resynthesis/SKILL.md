---
name: timbre-preserving-speech-resynthesis
description: Use when a task requires transforming an existing speech recording into
  a new utterance with reversed emotional tone (e.g. angry → happy) while keeping
  the speaker's voice, using AudioFab's toolchain.
when_to_use_negative:
- The task concerns music or general sound editing — this chain targets speech understanding/generation
  (FunTTS services), not Music MCP Server capabilities like Music Separation or Text2Music
- The goal is emotion analysis alone without resynthesis — the Emotion Tool / Speech
  Emotion Recognition service suffices without Whisper, Editing Tool, or CosyVoice2
- Local deployment of FunTTS models is unavailable and only the registered services
  in servers_config.json are active — AudioFab will not use models requiring local
  deployment
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the task, selects tools, and
    aggregates the chain into a single response
  repo: https://github.com/SmileHnu/AudioFab
- name: Emotion Tool (Speech Emotion Recognition)
  role: Classifies the emotional content of the input speech (e.g. 'happy' at 91%
    confidence) to establish the source emotion
- name: Whisper
  role: Transcribes the input speech to text for word-level manipulation
- name: Editing Tool
  role: Edits the transcript at word level, replacing emotion-bearing words with opposite-emotion
    counterparts
- name: CosyVoice2
  role: Synthesizes the edited transcript conditioned on the original speaker, preserving
    timbre (Text2Speech / Voice Conversion capability)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:21c5e10433012d0aa849eaf7f2f7edd827aff671e2f0ace7eb4427b9f230b7d4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# timbre-preserving-speech-resynthesis

## Summary

Reconstructs AudioFab's speech-perception use case (Figure 2, Case 2): chain the Emotion Tool (Speech Emotion Recognition), Whisper ASR, the Editing Tool, and CosyVoice2 to regenerate a speech clip with the opposite emotion while preserving the original speaker's timbre.

## When to use

Use when a task requires transforming an existing speech recording into a new utterance with reversed emotional tone (e.g. angry → happy) while keeping the speaker's voice, using AudioFab's toolchain.

## When NOT to use

- The task concerns music or general sound editing — this chain targets speech understanding/generation (FunTTS services), not Music MCP Server capabilities like Music Separation or Text2Music
- The goal is emotion analysis alone without resynthesis — the Emotion Tool / Speech Emotion Recognition service suffices without Whisper, Editing Tool, or CosyVoice2
- Local deployment of FunTTS models is unavailable and only the registered services in servers_config.json are active — AudioFab will not use models requiring local deployment

## Inputs

- Speech recording with an identifiable emotional tone (e.g. angry speech audio file)
- Natural-language user command describing the opposite-emotion resynthesis goal

## Outputs

- opposite_emotion_speech.wav — resynthesized speech with reversed emotion and preserved speaker timbre
- Emotion classification result (e.g. 'happy' at 91% confidence) from the Emotion Tool
- Word-level edited transcript produced by the Editing Tool
- Planner response confirming all subtasks executed successfully

## How to apply

Load the input speech recording (e.g. one with an angry tone, per the paper's video demo) and run the Emotion Tool to classify its emotional content, confirming reported-style outputs such as 'happy' at 91% confidence. Transcribe the speech to text with Whisper, then use the Editing Tool to rewrite emotion-bearing words at the word level into their opposite-emotion counterparts (e.g. rewriting 'Hello, I am good and feel happy' into its opposite-emotion variant). Resynthesize the edited transcript with CosyVoice2, conditioning on the original speaker so the timbre is preserved — mirroring the FunTTS Speech Generation capabilities Text2Speech and Voice Conversion. Write the result to opposite_emotion_speech.wav and let the LLM Planner verify that all subtasks executed successfully before generating the final response.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the task, selects tools, and aggregates the chain into a single response) — https://github.com/SmileHnu/AudioFab
- **Emotion Tool (Speech Emotion Recognition)** (Classifies the emotional content of the input speech (e.g. 'happy' at 91% confidence) to establish the source emotion)
- **Whisper** (Transcribes the input speech to text for word-level manipulation)
- **Editing Tool** (Edits the transcript at word level, replacing emotion-bearing words with opposite-emotion counterparts)
- **CosyVoice2** (Synthesizes the edited transcript conditioned on the original speaker, preserving timbre (Text2Speech / Voice Conversion capability))

## Examples

```
conda activate AudioFab; python scripts/start_all.py  # then issue: 'analyze this speech's emotion and regenerate it with the opposite sentiment'
```

## Evaluation signals

- Emotion Tool output on the input clip reports the expected style with a confidence figure (e.g. 'happy' at 91%)
- Whisper transcript matches the spoken content before editing
- Edited transcript contains opposite-emotion word replacements (e.g. 'happy' rewritten to its opposite-emotion variant)
- Resynthesized audio in opposite_emotion_speech.wav carries the opposite sentiment while the speaker's timbre remains recognizable (Voice Conversion conditioning)
- AudioFab's LLM Planner generates a complete response only after all subtasks have been successfully executed

## Limitations

- FunTTS models (including CosyVoice2) require local deployment or API configuration; without registration of FunTTS_mcp_servers in servers_config.json, this chain cannot run
- Local deployment of these models consumes substantial computing resources; the README recommends initially using models via API
- The workflow's success depends on the LLM Planner's toolchain evaluation and the two-stage tool-hallucination mitigation (intelligent filtering plus few-shot exemplar injection); errors in tool selection can break the chain

## Evidence

- [other] In the speech perception case, AudioFab chains four tools: Emotion Tool analyzes the emotion in the audio, Whisper transcribes the audio, Editing Tool edits the audio at the word level, and Cosy Voice2 synthesizes speech with the emotional tone.: [evidence span withheld — anchor status: not_found]
