---
name: music-style-description
description: Use when a natural-language audio task requires characterizing a song's
  musical style — e.g., 'understand the musical style of a song, extract its accompaniment,
  and create a new piece of music in the same style' — especially after Demucs has
  separated vocals from accompaniment.
when_to_use_negative:
- Input is the original full mix and the workflow also needs the vocal/accompaniment
  split — run Music Separation (Demucs) first so the style description is computed
  on the clean instrumental track.
- The request concerns speech understanding (ASR, speech emotion recognition, speaker
  diarization) or sound understanding (sound event detection) rather than music style
  — use the corresponding FunTTS capabilities instead.
- Local model deployment is unavailable and `FunTTS_mcp_servers` was removed from
  servers_config. — AudioFab will not use locally deployed models in that case, so
  the tool cannot execute.
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the task and whose MCP Server
    selects and invokes the Music Style Description tool
  repo: https://github.com/SmileHnu/AudioFab
- name: Qwen2-Audio
  role: Underlying model used by the FunTTS MCP Servers' Music Style Description tool
    to analyze the song's style
- name: Demucs
  role: Music Separation tool (Audio Separator MCP Servers) producing the accompaniment
    track that feeds style analysis
- name: DiffRhythm
  role: Music Generation tool (Music MCP Servers) that conditions on the analyzed
    style to synthesize new music
- name: Tool Query Servers
  role: Tool discovery/selection service that picks Music Style Description via listing,
    querying, and intelligent search
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:b7aa5877c015b0b6fe1a4fd12d6f915c547df92eeb2b85cfe217f70258b7f904
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
  - Tool Query Servers
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# music-style-description

## Summary

Generate a natural-language description of a song's musical style within the AudioFab agent framework, using the Music Style Description tool (Qwen2-Audio) exposed via the FunTTS MCP Servers. It is typically applied to a separated instrumental/accompaniment track so the style description can condition downstream music generation (DiffRhythm).

## When to use

Use when a natural-language audio task requires characterizing a song's musical style — e.g., 'understand the musical style of a song, extract its accompaniment, and create a new piece of music in the same style' — especially after Demucs has separated vocals from accompaniment.

## When NOT to use

- Input is the original full mix and the workflow also needs the vocal/accompaniment split — run Music Separation (Demucs) first so the style description is computed on the clean instrumental track.
- The request concerns speech understanding (ASR, speech emotion recognition, speaker diarization) or sound understanding (sound event detection) rather than music style — use the corresponding FunTTS capabilities instead.
- Local model deployment is unavailable and `FunTTS_mcp_servers` was removed from servers_config. — AudioFab will not use locally deployed models in that case, so the tool cannot execute.

## Inputs

- Separated instrumental/accompaniment track (audio file, produced by Demucs Music Separation via Audio Separator MCP Servers)
- Natural-language user request (e.g., understand the style, extract accompaniment, create new music in the same style) submitted through the AudioFab browser UI
- Deployed or API-token-accessible Qwen2-Audio model within the FunTTS MCP Servers

## Outputs

- Natural-language music style description (text artifact written to RESULT_FOLDER_PATH)
- Style-conditioning input consumed by DiffRhythm Music Generation in the Music MCP Servers

## How to apply

First ensure AudioFab is deployed: clone the repo, create the conda environment (`conda env create -f environment.yml` or `environment-lock.yml`), configure `.env` (LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME, MARKDOWN_FOLDER_PATH, RESULT_FOLDER_PATH) and `mcp_servers/servers_config.` with the local Python interpreter per service. Deploy or API-enable the FunTTS models (MD_TOKEN / HF_TOKEN in `servers/API_servers.py`) so `FunTTS_mcp_servers` remains registered in servers_config.. Submit the request in natural language; the LLM Planner decomposes the task and the MCP Server's Tool Selection service (Tool Query Servers) picks the Music Style Description tool by intelligent filtering of the tool list plus few-shot exemplar injection before invocation. Run the tool on the separated instrumental (accompaniment) track — running it on the full mix risks the style description being confounded by the vocal track, which is why AudioFab's Case 1 chains Demucs separation before Qwen2-Audio style analysis. The emitted style description is written to RESULT_FOLDER_PATH and is consumed by the Music MCP Servers' Music Generation (DiffRhythm) step to synthesize a new song conditioned on the accompaniment and the analyzed style.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the task and whose MCP Server selects and invokes the Music Style Description tool) — https://github.com/SmileHnu/AudioFab
- **Qwen2-Audio** (Underlying model used by the FunTTS MCP Servers' Music Style Description tool to analyze the song's style)
- **Demucs** (Music Separation tool (Audio Separator MCP Servers) producing the accompaniment track that feeds style analysis)
- **DiffRhythm** (Music Generation tool (Music MCP Servers) that conditions on the analyzed style to synthesize new music)
- **Tool Query Servers** (Tool discovery/selection service that picks Music Style Description via listing, querying, and intelligent search)

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- LLM Planner reports all subtasks (separation, style description, generation) as successfully executed before generating the final response.
- A style-description artifact exists in RESULT_FOLDER_PATH (file_exists check per artifact, alongside vocal, accompaniment, and generated-music files).
- The style description is derived from the separated accompaniment track, matching the README demo order: separate vocals from accompaniment, then analyze the musical style of the separated instrumental track.
- Tool selection trace shows the FunTTS Music Style Description tool was chosen after intelligent filtering of the tool list and few-shot exemplar injection, rather than hallucinated from the full library.
- Downstream Music Generation (DiffRhythm) accepts the style description and accompaniment as conditioning inputs without re-querying tool metadata.

## Limitations

- Running Qwen2-Audio and other FunTTS/Music/Audio Separator models locally consumes substantial computing resources; AudioFab's README recommends starting with API-based models (MD_TOKEN, HF_TOKEN).
- If local deployment is incomplete, the FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers registrations must be removed and those tools become unavailable.
- AudioFab carries the CC BY-NC 4.0 license (non-commercial), and the ACM MM paper link was not yet available in the official revised version at the time of the README.
- Existing audio agent setups suffer excessive context and tool hallucination when entire tool libraries are given to the LLM; the filtering + few-shot strategy mitigates but does not eliminate erroneous model calls.

## Evidence
