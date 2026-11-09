---
name: audio-source-separation
description: Use when a natural-language audio request requires isolating vocals from
  accompaniment — e.g. 'extract the accompaniment' or 'understand the style, extract
  the accompaniment, create new music in the same style' — as a prerequisite subtask
  for music analysis or generation.
when_to_use_negative:
- The audio is already a separated stem or accompaniment — re-separating adds no value;
  route to DSP/Audio Servers basic editing instead.
- The request needs only basic signal processing (format conversion, editing, feature
  extraction) without stem isolation — the DSP Servers cover these functions.
- FunTTS_mcp_servers, music_mcp_servers, or Audioseparator_mcp_servers have not been
  locally deployed or API-configured — the README instructs removing their registration,
  in which case separation is unavailable.
tools:
- name: AudioFab
  role: Agent framework whose LLM Planner decomposes the request and whose MCP Server
    dispatches the separation subtask; results are validated via file existence before
    the final response
  repo: https://github.com/SmileHnu/AudioFab
- name: Audio Separator MCP Servers
  role: MCP service that executes Music Separation (Demucs) to produce the vocal and
    accompaniment tracks
- name: Demucs
  role: Underlying music-separation model invoked for vocal/accompaniment splitting
    in the music creation case workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:9ba1faba7e8952032264a2219e4f5b4dad93ca950719144ac2bfd5a569ce3898
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - Audio Separator MCP Servers
  - AudioFab
  - Demucs
  - FunTTS MCP Servers
  - PyTorch
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# audio-source-separation

## Summary

Audio source separation within the AudioFab agent framework: an LLM-planned subtask in which the Audio Separator MCP Servers execute Music Separation (Demucs) to split an uploaded song into vocal and accompaniment tracks, which then feed downstream steps such as style description (Qwen2-Audio) or style-conditioned music generation (DiffRhythm).

## When to use

Use when a natural-language audio request requires isolating vocals from accompaniment — e.g. 'extract the accompaniment' or 'understand the style, extract the accompaniment, create new music in the same style' — as a prerequisite subtask for music analysis or generation.

## When NOT to use

- The audio is already a separated stem or accompaniment — re-separating adds no value; route to DSP/Audio Servers basic editing instead.
- The request needs only basic signal processing (format conversion, editing, feature extraction) without stem isolation — the DSP Servers cover these functions.
- FunTTS_mcp_servers, music_mcp_servers, or Audioseparator_mcp_servers have not been locally deployed or API-configured — the README instructs removing their registration, in which case separation is unavailable.

## Inputs

- an uploaded song (pop-song audio file) plus a natural-language task description via the AudioFab browser UI
- AudioSeparator_mcp_servers service registered in mcp_servers/servers_config.json (or HF_TOKEN in servers/API_servers.py for API-based use)

## Outputs

- vocal track written to RESULT_FOLDER_PATH
- accompaniment track written to RESULT_FOLDER_PATH

## How to apply

Deploy the AudioFab environment (conda env create -f environment.yml or environment-lock.yml, conda activate AudioFab), configure .env (LLM_API_KEY, MARKDOWN_FOLDER_PATH, RESULT_FOLDER_PATH) and mcp_servers/servers_config.json so each service's command points to the local Python interpreter, then launch with python scripts/start_all.py. Submit the song plus a natural-language request; the LLM Planner decomposes it into subtasks, and the MCP Server's Tool Selection service (Tool Query Servers) picks the Audio Separator MCP Servers by first streamlining the tool list via intelligent filtering and then injecting few-shot exemplars before invocation — a two-stage design that suppresses tool hallucination. The Audio Separator MCP Servers then execute Music Separation (Demucs) on the uploaded track, emitting a vocal track and an accompaniment track. Downstream, FunTTS MCP Servers run Music Style Description (Qwen2-Audio) on the separated instrumental track, so separation must complete first; the LLM Planner only generates the final response after all subtasks (including separation) succeed.

## Related tools

- **AudioFab** (Agent framework whose LLM Planner decomposes the request and whose MCP Server dispatches the separation subtask; results are validated via file existence before the final response) — https://github.com/SmileHnu/AudioFab
- **Audio Separator MCP Servers** (MCP service that executes Music Separation (Demucs) to produce the vocal and accompaniment tracks)
- **Demucs** (Underlying music-separation model invoked for vocal/accompaniment splitting in the music creation case workflow)

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- A vocal track and an accompaniment track both exist as artifacts in RESULT_FOLDER_PATH (file_exists check per artifact).
- The LLM Planner reports the separation subtask as successfully executed before generating the final response.
- Separation completes before the Music Style Description step runs on the separated instrumental track.
- The few-shot-exemplar-guided tool selection chose the Audio Separator MCP Servers rather than a hallucinated tool call.
- Downstream DiffRhythm generation conditions on the accompaniment and analyzed style, implying the separation artifacts were well-formed.

## Limitations

- Local deployment of the separation models consumes substantial computing resources; AudioFab recommends starting with API tokens (MD_TOKEN, HF_TOKEN) instead of full local deployment.
- If Audioseparator_mcp_servers is not registered (un-deployed models), AudioFab will not run the separation service at all.
- Each MCP tool runs in its own dedicated dependency environment to avoid conflicts; incorrect 'command' paths in servers_config.json break service launch (verify with scripts/check.sh).

## Evidence

- [readme] Upload a song to separate the vocals from the accompaniment. Subsequently, analyze the musical style of the separated instrumental track to provide a basis for future music creation or adaptation.: "Upload a song to separate the vocals from the accompaniment. Subsequently, analyze the musical style of the separated instrumental track to provide a basis for future music creation or adaptation."
- [readme] This architecture enables each tool to run in its own dedicated dependency environment, fundamentally eliminating dependency conflicts.: "This architecture enables each tool to run in its own dedicated dependency environment, fundamentally eliminating dependency conflicts."
- [methods] first, it streamlines the tool list via intelligent filtering to shorten the context; second, it injects few-shot exemplars for precise guidance before invocation: "first, it streamlines the tool list via intelligent filtering to shorten the context; second, it injects few-shot exemplars for precise guidance before invocation"
- [results] AudioFab chains three tools: Qwen2-Audio analyzes the song's style, Demucs separates vocals and accompaniment: [evidence span withheld — anchor status: not_found]
