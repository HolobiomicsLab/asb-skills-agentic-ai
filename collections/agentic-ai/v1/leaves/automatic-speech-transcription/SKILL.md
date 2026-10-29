---
name: automatic-speech-transcription
description: Use when a speech recording (e.g. .wav with an emotional tone) must be
  converted to a text transcript as an intermediate step, such as before word-level
  editing and emotion-reversed resynthesis in AudioFab.
when_to_use_negative:
- Input audio contains music, sound events, or ambient scenes rather than speech —
  use Music Understanding / Sound Event Detection tools in the FunTTS or Music MCP
  Servers instead
- The content is already available as text or a Markdown/TXT/JSON file — the Markdown
  Servers handle those formats without ASR
- The goal requires only acoustic attributes (e.g. emotion classification at 91% confidence)
  with no need for the spoken words — the Emotion Tool alone suffices
tools:
- name: Whisper
  role: ASR model that transcribes the input speech to text within the AudioFab chain
  repo: https://github.com/SmileHnu/AudioFab
- name: AudioFab
  role: Agent framework whose LLM Planner and MCP Server select and invoke the ASR
    tool
  repo: https://github.com/SmileHnu/AudioFab
- name: Editing Tool
  role: Consumes the Whisper transcript to perform word-level editing of emotion-bearing
    words
  repo: https://github.com/SmileHnu/AudioFab
- name: CosyVoice2
  role: Resynthesizes the edited transcript conditioned on the original speaker (Text2Speech/Voice
    Conversion)
  repo: https://github.com/SmileHnu/AudioFab
- name: Emotion Tool
  role: Upstream Speech Emotion Recognition step that classifies the input clip's
    emotion before transcription in the case workflow
  repo: https://github.com/SmileHnu/AudioFab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:5b6b089d4e1e3f64a2a5dba61c66a51611c5ade3f1cc96b323b98081570b2558
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - CosyVoice2
  - Editing Tool
  - Emotion Tool
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

# automatic-speech-transcription

## Summary

Transcribes speech recordings to text using Whisper as served by AudioFab's FunTTS MCP Servers (ASR under Speech Understanding), providing the text substrate for downstream word-level editing and Text2Speech/Voice-Conversion resynthesis.

## When to use

Use when a speech recording (e.g. .wav with an emotional tone) must be converted to a text transcript as an intermediate step, such as before word-level editing and emotion-reversed resynthesis in AudioFab.

## When NOT to use

- Input audio contains music, sound events, or ambient scenes rather than speech — use Music Understanding / Sound Event Detection tools in the FunTTS or Music MCP Servers instead
- The content is already available as text or a Markdown/TXT/JSON file — the Markdown Servers handle those formats without ASR
- The goal requires only acoustic attributes (e.g. emotion classification at 91% confidence) with no need for the spoken words — the Emotion Tool alone suffices

## Inputs

- Speech recording audio file (e.g. .wav speech clip with an emotional tone such as angry)
- Natural-language user command describing the transcription task

## Outputs

- Plain-text transcript of the spoken words (e.g. 'Hello, I am good and feel happy')
- Transcript passed downstream to the Editing Tool for word-level emotion rewriting

## How to apply

Submit the input speech recording to AudioFab via its natural-language interface; the LLM Planner decomposes the task and directs the MCP Server to invoke the ASR tool (Whisper) from the FunTTS MCP Servers' Speech Understanding service. Whisper returns a plain-text transcript of the spoken words (e.g. 'Hello, I am good and feel happy'). Because AudioFab streamlines its tool list via intelligent filtering and injects few-shot exemplars before invocation, the planner should select the ASR tool specifically for speech input rather than music or sound-event tools. The transcript then feeds the Editing Tool for word-level replacement (rewriting emotion-bearing words into their opposite-emotion variants), and CosyVoice2 resynthesizes the edited transcript conditioned on the original speaker to preserve timbre. Rationale: text is the editable intermediate representation that lets a fixed utterance be re-generated with flipped emotion while keeping speaker identity.

## Related tools

- **Whisper** (ASR model that transcribes the input speech to text within the AudioFab chain) — https://github.com/SmileHnu/AudioFab
- **AudioFab** (Agent framework whose LLM Planner and MCP Server select and invoke the ASR tool) — https://github.com/SmileHnu/AudioFab
- **Editing Tool** (Consumes the Whisper transcript to perform word-level editing of emotion-bearing words) — https://github.com/SmileHnu/AudioFab
- **CosyVoice2** (Resynthesizes the edited transcript conditioned on the original speaker (Text2Speech/Voice Conversion)) — https://github.com/SmileHnu/AudioFab
- **Emotion Tool** (Upstream Speech Emotion Recognition step that classifies the input clip's emotion before transcription in the case workflow) — https://github.com/SmileHnu/AudioFab

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- Transcript matches the words actually spoken in the input clip (e.g. 'Hello, I am good and feel happy' for the demo speech)
- The AudioFab planner reports all subtasks in the toolchain executed successfully, including the Whisper ASR subtask
- Downstream word-level edits land on real transcript words, confirming transcription fidelity (mis-transcriptions would break the opposite-emotion rewrite)
- Resynthesized output (opposite_emotion_speech.wav) is intelligible speech consistent with the edited transcript and preserves the original speaker timbre
- Emotion analysis of the final audio confirms the emotion was reversed relative to the input (e.g. angry input yielding happy-style output)

## Limitations

- ASR in AudioFab depends on FunTTS MCP Server models that may require local deployment; if not deployed, those service registrations must be removed from servers_config.json and AudioFab cannot use them
- Local deployment of models consumes substantial computing resources; the README recommends using some models via API (MD_TOKEN, HF_TOKEN) for a quicker experience
- Transcription quality is not quantified in the paper; noisy or heavily emotional speech may degrade transcript accuracy and corrupt word-level edits

## Evidence

- [full_text] Whisper transcribes the audio: [evidence span withheld — anchor status: not_found]
