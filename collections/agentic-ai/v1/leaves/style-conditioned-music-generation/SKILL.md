---
name: style-conditioned-music-generation
description: Use when a reference song (e.g., a pop track) is provided with a natural-language
  request to understand its musical style, extract its accompaniment, and create new
  music in the same style.
when_to_use_negative:
- The task involves only speech or sound understanding (e.g., ASR, speech emotion
  recognition) with no music generation or style transfer component.
- FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers have been
  removed from servers_config.json because local models were not deployed and no API
  tokens (MD_TOKEN, HF_TOKEN) are configured — the required tools will not be available.
- The input is already a finished composition request needing only Text2Music/Lyrics2Song
  without a reference track for style conditioning.
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the request and whose Tool Query
    Servers select and invoke the toolchain
  repo: https://github.com/SmileHnu/AudioFab
- name: Demucs
  role: Executed by the Audio Separator MCP Servers as Music Separation to produce
    the vocal and accompaniment tracks
- name: Qwen2-Audio
  role: Executed by the FunTTS MCP Servers as Music Style Description on the separated
    instrumental track to emit the style description
- name: DiffRhythm
  role: Executed by the Music MCP Servers as Music Generation, conditioning on the
    accompaniment and analyzed style to synthesize the new music segment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:f949b0c95c513d980252e13dfcf933be616860a3876d50e2eff20e67a257197b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - Demucs
  - DiffRhythm
  - FunTTS MCP Servers
  - Music MCP Servers
  - PyTorch
  - Python
  - Qwen2-Audio
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# style-conditioned-music-generation

## Summary

An AudioFab workflow that generates a new piece of music in the style of a reference song by chaining Demucs vocal/accompaniment separation, Qwen2-Audio style description of the instrumental track, and DiffRhythm style-conditioned generation.

## When to use

Use when a reference song (e.g., a pop track) is provided with a natural-language request to understand its musical style, extract its accompaniment, and create new music in the same style.

## When NOT to use

- The task involves only speech or sound understanding (e.g., ASR, speech emotion recognition) with no music generation or style transfer component.
- FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers have been removed from servers_config.json because local models were not deployed and no API tokens (MD_TOKEN, HF_TOKEN) are configured — the required tools will not be available.
- The input is already a finished composition request needing only Text2Music/Lyrics2Song without a reference track for style conditioning.

## Inputs

- Pop song / reference audio track uploaded through the AudioFab browser UI
- Natural-language task description (style understanding + accompaniment extraction + same-style music creation)
- Configured .env (LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME, RESULT_FOLDER_PATH) and mcp_servers/servers_config.json with FunTTS, Music, and Audio Separator services registered

## Outputs

- Vocal track (from Demucs Music Separation)
- Accompaniment track (from Demucs Music Separation)
- Style description of the instrumental track (from Qwen2-Audio Music Style Description)
- Generated music segment conditioned on accompaniment + style (from DiffRhythm Music Generation), written to RESULT_FOLDER_PATH

## How to apply

Submit the song with a natural-language request (understand the style, extract the accompaniment, create new music in the same style); the LLM Planner decomposes it into subtasks and the MCP Server's Tool Selection service (Tool Query Servers) picks tools via intelligent filtering of the tool list plus few-shot exemplar injection. The Audio Separator MCP Servers run Music Separation (Demucs) to emit the vocal track and accompaniment track. The FunTTS MCP Servers run Music Style Description (Qwen2-Audio) on the separated instrumental track to emit the style description. The Music MCP Servers run Music Generation (DiffRhythm), conditioning on the accompaniment and the analyzed style to synthesize the new music segment. Rationale: the two-stage tool-selection strategy (intelligent filtering, then few-shot exemplars) suppresses tool hallucination, and chaining separation before style analysis grounds the style description in the instrumental track only.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the request and whose Tool Query Servers select and invoke the toolchain) — https://github.com/SmileHnu/AudioFab
- **Demucs** (Executed by the Audio Separator MCP Servers as Music Separation to produce the vocal and accompaniment tracks)
- **Qwen2-Audio** (Executed by the FunTTS MCP Servers as Music Style Description on the separated instrumental track to emit the style description)
- **DiffRhythm** (Executed by the Music MCP Servers as Music Generation, conditioning on the accompaniment and analyzed style to synthesize the new music segment)

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- The LLM Planner reports all subtasks as successfully executed before generating the final response
- Accompaniment, vocal, style-description, and generated-music artifacts all exist in RESULT_FOLDER_PATH (file_exists check per artifact)
- The vocal and accompaniment tracks are non-empty audio outputs of the Demucs separation step, ordered before style analysis in the toolchain
- The style description is derived from the separated instrumental track (not the original full mix), per the demo scenario
- The generated music segment is conditioned on both the accompaniment track and the Qwen2-Audio style description

## Limitations

- Music generation tools require local deployment of models (substantial computing resources) or API tokens; if neither is configured, AudioFab will not use these models and the workflow cannot complete
- AudioFab's music-generation coverage is bounded by what its MCP services register (Music2Song, Text2Music, Lyrics2Song via Music MCP Servers; Music Separation via Audio Separator MCP Servers)
- Deploying all required models locally consumes substantial computing resources; the README recommends starting with API-based models

## Evidence
