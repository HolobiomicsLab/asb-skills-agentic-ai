---
name: mcp-tool-orchestration
description: Use when a natural-language audio request must be decomposed into subtasks
  and routed to MCP-registered tools (DSP, FunTTS, Music, Audio Separator) through
  AudioFab's four-stage loop with anti-hallucination tool filtering.
when_to_use_negative:
- The task is a single-function tool call (e.g., one separation or one ASR pass) that
  needs no task decomposition, tool selection, or chained multi-tool execution
- Local compute is insufficient for the FunTTS/Music/Audio Separator models and no
  MD_TOKEN/HF_TOKEN API access is configured — those services must be deregistered
  and the skill cannot complete advanced cases
- Commercial or for-profit use is intended — AudioFab is released under the CC BY-NC
  4.0 license
tools:
- name: AudioFab
  role: Agent framework hosting the orchestration loop; installed via Conda and launched
    with scripts/start_all.py, with a browser UI for natural-language commands
  repo: https://github.com/SmileHnu/AudioFab
- name: LLM Planner
  role: Decomposes the user command into ordered subtasks, directs tool invocation,
    evaluates toolchains from Tool Selection, and generates the final response after
    all subtasks succeed
- name: MCP Client
  role: Receives user commands and forwards them to the LLM Planner, sends tool invocation
    requests to the MCP Server, and delivers the final response
- name: MCP Server
  role: Uniformly manages and executes the integrated tools across 9 services and
    provides the Tool Selection service
- name: Tool Query Servers
  role: Tool discovery and information queries — selects the most suitable tool via
    listing, querying, and intelligent search over the tool JSON
- name: Python
  role: Runtime interpreter whose local path must replace each service's 'command'
    entry in servers_config.
- name: Conda
  role: Environment creation via environment.yml or environment-lock.yml and activation
    with 'conda activate AudioFab'
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_007
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:ed4c7a5bc19f8e2407098e85d56c554beaa405abbe759f1c7c11fa3b06efccb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - Conda
  - FunTTS MCP Servers
  - Hallo2
  - LLM Planner
  - MCP Client
  - MCP Server
  - PyTorch
  - Python
  - Tool Query Servers
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# mcp-tool-orchestration

## Summary

Orchestrates AudioFab's MCP-based tool learning control loop, in which an LLM Planner decomposes a natural-language audio command into ordered subtasks and a Tool Selection service suppresses tool hallucination via intelligent filtering plus few-shot exemplar injection. Each selected tool then executes on an MCP Server in its own dedicated dependency environment, resolving the dependency conflicts that plague unified audio workflows.

## When to use

Use when a natural-language audio request must be decomposed into subtasks and routed to MCP-registered tools (DSP, FunTTS, Music, Audio Separator) through AudioFab's four-stage loop with anti-hallucination tool filtering.

## When NOT to use

- The task is a single-function tool call (e.g., one separation or one ASR pass) that needs no task decomposition, tool selection, or chained multi-tool execution
- Local compute is insufficient for the FunTTS/Music/Audio Separator models and no MD_TOKEN/HF_TOKEN API access is configured — those services must be deregistered and the skill cannot complete advanced cases
- Commercial or for-profit use is intended — AudioFab is released under the CC BY-NC 4.0 license

## Inputs

- Natural-language user command with uploaded audio (e.g., a pop song, angry-tone speech recording, or portrait image)
- mcp_servers/servers_config. declaring each service's Python interpreter 'command' and server script 'args'
- .env configuration with LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME, MARKDOWN_FOLDER_PATH, RESULT_FOLDER_PATH
- Tool inventory JSON (tool names, descriptions, examples) documented in ToolKits.md
- MD_TOKEN and HF_TOKEN in servers/API_servers.py when models are used via API instead of local deployment

## Outputs

- Ordered subtask plan produced by the LLM Planner
- Selected toolchain with per-tool parameters from the Tool Selection service
- Tool execution results returned to the planner for subsequent decisions
- Final natural-language response delivered to the user by the MCP Client
- Artifact files (vocal/accompaniment tracks, style description, generated music, MP4 video) written to RESULT_FOLDER_PATH

## How to apply

Clone https://github.com/SmileHnu/AudioFab.git, build the environment with 'conda env create -f environment.yml' (or environment-lock.yml) and 'conda activate AudioFab', then edit the .env (LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME, MARKDOWN_FOLDER_PATH, RESULT_FOLDER_PATH) and mcp_servers/servers_config. so each service's 'command' points to your Python interpreter, verifying with scripts/check.sh. Deploy the FunTTS, Music, and Audio Separator models locally per ToolKitsGuide.md or add MD_TOKEN/HF_TOKEN to servers/API_servers.py; if local deployment is incomplete, deregister FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers in servers_config.. Launch with 'python scripts/start_all.py', open the browser UI from the command-line prompts, and submit one natural-language command (e.g., understand a song's musical style, extract its accompaniment, and create a new piece in the same style). The LLM Planner decomposes the command into ordered subtasks; the MCP Server's Tool Selection (Tool Query Servers) enumerates tools over the tool JSON and applies the two-stage anti-hallucination strategy — first streamlining the tool list via intelligent filtering to shorten the context, then injecting few-shot exemplars before invocation. The MCP Client sends invocation requests to the MCP Server, each selected tool executes in its own dedicated dependency environment, and results return to the planner for the next decision. Response generation occurs only after all subtasks report success, and validation requires produced artifacts (vocal/accompaniment tracks, style description, generated music or MP4) to exist in RESULT_FOLDER_PATH.

## Related tools

- **AudioFab** (Agent framework hosting the orchestration loop; installed via Conda and launched with scripts/start_all.py, with a browser UI for natural-language commands) — https://github.com/SmileHnu/AudioFab
- **LLM Planner** (Decomposes the user command into ordered subtasks, directs tool invocation, evaluates toolchains from Tool Selection, and generates the final response after all subtasks succeed)
- **MCP Client** (Receives user commands and forwards them to the LLM Planner, sends tool invocation requests to the MCP Server, and delivers the final response)
- **MCP Server** (Uniformly manages and executes the integrated tools across 9 services and provides the Tool Selection service)
- **Tool Query Servers** (Tool discovery and information queries — selects the most suitable tool via listing, querying, and intelligent search over the tool JSON)
- **Python** (Runtime interpreter whose local path must replace each service's 'command' entry in servers_config.)
- **Conda** (Environment creation via environment.yml or environment-lock.yml and activation with 'conda activate AudioFab')

## Examples

```
conda activate AudioFab && python scripts/start_all.py
```

## Evaluation signals

- scripts/check.sh runs clean, confirming servers_config. interpreter paths and .env entries are correctly configured
- The LLM Planner reports every subtask as successfully executed before emitting the final response (no response on partial success)
- Each expected artifact (vocal track, accompaniment track, style description, generated music, MP4) exists in RESULT_FOLDER_PATH via a file_exists check per artifact
- One natural-language command completes planning → tool selection → tool call/execution → response generation end-to-end
- The tool list is streamlined by intelligent filtering before few-shot exemplar injection, and no erroneous (hallucinated) tool calls occur

## Limitations

- Loading an entire tool library into the LLM causes excessive contextual information and tool hallucination, so the two-stage filtering must be configured correctly or invocation stability degrades
- Local deployment of the FunTTS, Music, and Audio Separator models consumes substantial computing resources; the README recommends starting with API models (MD_TOKEN, HF_TOKEN) for a quicker experience
- If local deployment is incomplete, FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers must be removed from servers_config. or AudioFab will not use those models

## Evidence

- [intro] existing systems burden LLMs with entire tool libraries, resulting in excessive contextual information and increasing the risk of interpretive errors (aka tool hallucination): [evidence span withheld — anchor status: not_found]
- [readme] Edit `mcp_servers/servers_config.` to match your local setup: replace the `command` under each service with your Python interpreter path.: [evidence span withheld — anchor status: not_found]
