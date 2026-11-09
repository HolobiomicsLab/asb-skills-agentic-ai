---
name: audio-toolchain-orchestration
description: Use when a task requires transforming an uploaded speech recording's
  emotional tone (e.g. angry to happy) while keeping the speaker's voice, needing
  chained recognition-transcription-editing-resynthesis steps rather than a single
  tool.
when_to_use_negative:
- The task targets music (e.g. vocal/accompaniment separation or music style analysis)
  rather than speech emotion transformation — use the Music MCP Servers / Audio Separator
  workflow instead
- Only a single tool suffices (e.g. plain ASR or plain Text2Speech) — the two-stage
  intelligent filtering and chain overhead is unnecessary
- Local models like CosyVoice2 are not deployed and FunTTS_mcp_servers is unregistered;
  AudioFab will not use models requiring local deployment
edam_operation: http://edamontology.org/operation_3280
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the command, selects and invokes
    tools via the MCP Server, and generates the final response
  repo: https://github.com/SmileHnu/AudioFab
- name: Emotion Tool
  role: Speech Emotion Recognition backend that classifies the input clip's emotional
    content before rewriting
- name: Whisper
  role: Transcribes the input speech to text for word-level editing
- name: Editing Tool
  role: Edits the transcript at the word level, replacing emotion-bearing words with
    opposite-emotion counterparts
- name: CosyVoice2
  role: Synthesizes the edited transcript with the emotional tone while conditioning
    on the original speaker to preserve timbre
- name: FunTTS MCP Servers
  role: MCP service exposing Speech Understanding (SER, ASR) and Speech Generation
    (Text2Speech, Voice Conversion) tools used in the chain
  repo: https://github.com/SmileHnu/AudioFab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:8da186ae1a6e441c1edab2c291f95e797e2ea75ae4c17193703515a31fbfd4ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - CosyVoice2
  - Editing Tool
  - Emotion Tool
  - FunTTS MCP Servers
  - Whisper
  edam_operation: http://edamontology.org/operation_3280
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# audio-toolchain-orchestration

## Summary

Chain AudioFab's LLM Planner, MCP Server, and specialized audio tools (Emotion Tool, Whisper, Editing Tool, CosyVoice2) into a multi-stage pipeline that transforms a speech clip's emotion while preserving speaker timbre, as demonstrated in AudioFab's speech perception case.

## When to use

Use when a task requires transforming an uploaded speech recording's emotional tone (e.g. angry to happy) while keeping the speaker's voice, needing chained recognition-transcription-editing-resynthesis steps rather than a single tool.

## When NOT to use

- The task targets music (e.g. vocal/accompaniment separation or music style analysis) rather than speech emotion transformation — use the Music MCP Servers / Audio Separator workflow instead
- Only a single tool suffices (e.g. plain ASR or plain Text2Speech) — the two-stage intelligent filtering and chain overhead is unnecessary
- Local models like CosyVoice2 are not deployed and FunTTS_mcp_servers is unregistered; AudioFab will not use models requiring local deployment

## Inputs

- Input speech recording (e.g. WAV clip with an angry tone, per the paper's video demo)
- Natural-language user command describing the desired emotion reversal
- MCP server configuration (mcp_servers/servers_config.json) with FunTTS MCP Servers registered

## Outputs

- opposite_emotion_speech.wav containing resynthesized speech with the reversed emotion and preserved speaker timbre
- Emotion classification result (e.g. 'happy' at 91% confidence)
- Word-level edited transcript (e.g. opposite-emotion rewrite of 'Hello, I am good and feel happy')
- Planner response generated after all subtasks executed successfully

## How to apply

Decompose the user command into subtasks via the LLM Planner, then let the MCP Server's Tool Query Servers intelligently filter the tool list and inject few-shot exemplars before invocation to suppress tool hallucination. First run the Emotion Tool (Speech Emotion Recognition) on the input recording to classify its emotion, e.g. 'happy' at 91% confidence. Second, transcribe the speech with Whisper, then use the Editing Tool to rewrite emotion-bearing words at the word level (e.g. 'Hello, I am good and feel happy' into its opposite-emotion variant). Finally, resynthesize the edited transcript with CosyVoice2 conditioned on the original speaker so timbre is preserved, mirroring the FunTTS MCP Servers' Text2Speech and Voice Conversion generation capabilities. Write the result to opposite_emotion_speech.wav and confirm the planner reports all subtasks executed successfully before generating the response.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the command, selects and invokes tools via the MCP Server, and generates the final response) — https://github.com/SmileHnu/AudioFab
- **Emotion Tool** (Speech Emotion Recognition backend that classifies the input clip's emotional content before rewriting)
- **Whisper** (Transcribes the input speech to text for word-level editing)
- **Editing Tool** (Edits the transcript at the word level, replacing emotion-bearing words with opposite-emotion counterparts)
- **CosyVoice2** (Synthesizes the edited transcript with the emotional tone while conditioning on the original speaker to preserve timbre)
- **FunTTS MCP Servers** (MCP service exposing Speech Understanding (SER, ASR) and Speech Generation (Text2Speech, Voice Conversion) tools used in the chain) — https://github.com/SmileHnu/AudioFab

## Examples

```
conda activate AudioFab && python scripts/start_all.py  # then: "analyze this angry recording's emotion and resynthesize it with the opposite emotion preserving the speaker's timbre"
```

## Evaluation signals

- Emotion Tool returns a confident style label on the input clip (paper demo reports 'happy' at 91% confidence)
- Whisper transcript matches the spoken words of the input recording before editing
- Edited transcript contains opposite-emotion word replacements and no other content changes
- Resynthesized audio preserves the original speaker's timbre while exhibiting the opposite emotion
- AudioFab LLM Planner reports all subtasks executed successfully and the output file opposite_emotion_speech.wav exists

## Limitations

- Local deployment of models like CosyVoice2 is complex and consumes substantial computing resources; the README recommends using models via API initially
- If FunTTS_mcp_servers, music_mcp_servers, or Audioseparator_mcp_servers are not registered in servers_config.json, AudioFab will not use models that require local deployment
- Existing systems burdening LLMs with entire tool libraries risk tool hallucination; AudioFab mitigates but does not eliminate this via intelligent filtering and few-shot exemplars

## Evidence

- [full_text] Emotion Tool analyzes the emotion in the audio, Whisper transcribes the audio, Editing Tool edits the audio at the word level, and Cosy Voice2 synthesizes speech with the emotional tone.: [evidence span withheld — anchor status: not_found]
