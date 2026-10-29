---
name: portrait-image-preprocessing-for-video-generation
description: Use when the task pairs an uploaded portrait image with a speech audio
  file and the goal is a lip-synced talking-head video driven by that audio inside
  the AudioFab Multimodal Processing (Case 3) pipeline.
when_to_use_negative:
- The task is audio-only (e.g. DSP, separation, TTS) with no image input or talking-head
  goal — use DSP Servers or Audio Separator MCP Servers instead.
- FunTTS/Music/Audio Separator MCP Servers are not locally deployed and no API tokens
  are configured — per the README, their registrations should be removed and such
  advanced capabilities will be unavailable.
edam_operation: http://edamontology.org/operation_3359
tools:
- name: AudioFab
  role: Agent framework whose MCP Client, LLM Planner, and MCP Server plan, select,
    and invoke the image and animation tools
  repo: https://github.com/SmileHnu/AudioFab
- name: Hallo2
  role: Talking-head model exposed via the Speech2Talking-head capability of the Music
    MCP Servers; drives image animation from the preprocessed audio
- name: PyTorch
  role: Backend dependency for the deployed models; Tensor Servers operate on PyTorch
    tensors and NumPy arrays
  repo: https://github.com/SmileHnu/AudioFab
- name: Conda
  role: One-command environment setup (environment.yml or environment-lock.yml) for
    the AudioFab runtime
  repo: https://github.com/SmileHnu/AudioFab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:c91c3a79c561b71a933206c960b3923c29cd415eadd0aec7b4e90d58cca032ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - Conda
  - Hallo2
  - PyTorch
  - Python
  edam_operation: http://edamontology.org/operation_3359
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# portrait-image-preprocessing-for-video-generation

## Summary

Prepares a source portrait image (resize/normalize, scale, resolution, format) so it can be driven into a talking-head animation by an input speech audio via AudioFab's Image Tools and the Hallo2 model, then exported as an MP4.

## When to use

Use when the task pairs an uploaded portrait image with a speech audio file and the goal is a lip-synced talking-head video driven by that audio inside the AudioFab Multimodal Processing (Case 3) pipeline.

## When NOT to use

- The task is audio-only (e.g. DSP, separation, TTS) with no image input or talking-head goal — use DSP Servers or Audio Separator MCP Servers instead.
- FunTTS/Music/Audio Separator MCP Servers are not locally deployed and no API tokens are configured — per the README, their registrations should be removed and such advanced capabilities will be unavailable.

## Inputs

- Uploaded source portrait image (image file)
- Uploaded speech audio file
- servers_config.json with per-service Python interpreter paths
- LLM_API_KEY / LLM_BASE_URL / LLM_MODEL_NAME settings in .env

## Outputs

- MP4 talking-head video in the configured RESULT_FOLDER_PATH, lip-synced to the input audio
- LLM Planner response summarizing executed subtasks

## How to apply

Register the task through the AudioFab MCP Client so the LLM Planner decomposes it; the planner streamlines the tool list via intelligent filtering and injects few-shot exemplars before invocation to suppress tool hallucination. First preprocess the audio with DSP Servers (feature extraction and format conversion) so it is a suitable animation-driving signal. Next run Image Tools on the portrait to fix scale, resolution, and format (resize/normalize) for the animation model. Invoke Hallo2 through the Music MCP Servers' Speech2Talking-head capability to generate talking-head frames from the preprocessed audio, then use the Save Tool to assemble frames with the audio into an MP4 in RESULT_FOLDER_PATH. Advanced servers (e.g. music_mcp_servers) must be deployed locally or accessed via API, with MD_TOKEN/HF_TOKEN configured in servers/API_servers.py.

## Related tools

- **AudioFab** (Agent framework whose MCP Client, LLM Planner, and MCP Server plan, select, and invoke the image and animation tools) — https://github.com/SmileHnu/AudioFab
- **Hallo2** (Talking-head model exposed via the Speech2Talking-head capability of the Music MCP Servers; drives image animation from the preprocessed audio)
- **PyTorch** (Backend dependency for the deployed models; Tensor Servers operate on PyTorch tensors and NumPy arrays) — https://github.com/SmileHnu/AudioFab
- **Conda** (One-command environment setup (environment.yml or environment-lock.yml) for the AudioFab runtime) — https://github.com/SmileHnu/AudioFab

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- Output MP4 file exists in RESULT_FOLDER_PATH and is a playable video file.
- The video corresponds to a completely new, lip-synced video driven by the input speech audio (not the original still image).
- The LLM Planner's toolchain passed Tool Selection evaluation, with the tool list narrowed by intelligent filtering and few-shot exemplars injected before Hallo2 invocation.
- All subtasks report successful execution before the LLM Planner generates the final response.

## Limitations

- Hallo2 and other advanced models consume substantial computing resources when deployed locally; the README recommends starting via API (MD_TOKEN, HF_TOKEN in servers/API_servers.py).
- If local deployment is incomplete, music_mcp_servers and related services must be unregistered in servers_config.json or AudioFab will attempt to use unavailable models.
- The paper's official link was pending at curation time; index and behavior were verified against the GitHub README (dev1.1 branch updated retrieval model and ranking algorithm).

## Evidence

- [abstract] Hallo2 drives image animation using the audio, and Save Tool saves the generated output as an MP4 file: [evidence span withheld — anchor status: not_found]
