---
name: llm-task-decomposition-and-tool-selection
description: Use when a natural-language user command must be broken into executable
  subtasks and mapped onto a large tool library, especially when loading the whole
  tool list risks tool hallucination or excessive context.
when_to_use_negative:
- The task is a single-function operation directly matching one registered tool, where
  decomposition and tool filtering add overhead without benefit
- Models requiring local deployment (FunTTS, Music, Audio Separator services) are
  not configured and were de-registered in servers_config., so selected tools cannot
  be invoked
- You need free-form reasoning over an arbitrary API rather than coordinated tool
  execution by an LLM agent
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: AudioFab
  role: Agent framework hosting the LLM Planner, MCP Client, and MCP Server that execute
    decomposition and tool selection
  repo: https://github.com/SmileHnu/AudioFab
- name: Tool Query Servers
  role: MCP service performing tool discovery, listing, querying, and intelligent
    search to pick the most suitable tool
  repo: https://github.com/SmileHnu/AudioFab
- name: Python
  role: Runtime for launching the agent (scripts/start_all.py) and for MCP Server
    service scripts
- name: Conda
  role: Environment setup via environment.yml / environment-lock.yml before running
    the planner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:d2ffc5ba0da7aaad619fa7c29722152ba6513ef5dc6f4281b1b26be670a6a4f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - Conda
  - FunTTS MCP Servers
  - PyTorch
  - Python
  - Tool Query Servers
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# llm-task-decomposition-and-tool-selection

## Summary

Decompose a natural-language audio task into subtasks and select the right tools via a two-stage strategy (intelligent filtering of the tool list plus few-shot exemplar injection), as implemented by AudioFab's LLM Planner and MCP Server Tool Query Servers. This prevents tool hallucination from oversized tool contexts.

## When to use

Use when a natural-language user command must be broken into executable subtasks and mapped onto a large tool library, especially when loading the whole tool list risks tool hallucination or excessive context.

## When NOT to use

- The task is a single-function operation directly matching one registered tool, where decomposition and tool filtering add overhead without benefit
- Models requiring local deployment (FunTTS, Music, Audio Separator services) are not configured and were de-registered in servers_config., so selected tools cannot be invoked
- You need free-form reasoning over an arbitrary API rather than coordinated tool execution by an LLM agent

## Inputs

- Natural-language user command describing a complex multi-stage audio task (e.g., 'understand the style, extract the accompaniment, create new music in the same style')
- Uploaded audio files (e.g., a pop song)
- Registered tool list from the MCP Server Tool Kits (FunTTS, Music, Audio Separator MCP Servers, plus Markdown/Tensor/DSP/Audio services)

## Outputs

- Decomposed subtask plan with per-subtask tool selections
- Tool execution results forwarded back to the LLM Planner (e.g., vocal/accompaniment tracks, style description, generated music in RESULT_FOLDER_PATH)
- Complete final response to the user after all subtasks succeed

## How to apply

First, the LLM Planner performs task decomposition and planning based on the user command, splitting it into ordered subtasks (e.g., style description via Qwen2-Audio, music separation via Demucs, music generation via DiffRhythm). Second, tool selection is delegated to the MCP Server's Tool Selection service: Tool Query Servers streamline the tool list via intelligent filtering to shorten the context, then inject few-shot exemplars for precise guidance before invocation. The LLM Planner directs the MCP Server to invoke the selected tools, evaluates the returned toolchains, and makes subsequent decisions from execution results. Only after all subtasks have been successfully executed does the Planner generate the complete response; each new tool enters via a simple registration process into the MCP Server.

## Related tools

- **AudioFab** (Agent framework hosting the LLM Planner, MCP Client, and MCP Server that execute decomposition and tool selection) — https://github.com/SmileHnu/AudioFab
- **Tool Query Servers** (MCP service performing tool discovery, listing, querying, and intelligent search to pick the most suitable tool) — https://github.com/SmileHnu/AudioFab
- **Python** (Runtime for launching the agent (scripts/start_all.py) and for MCP Server service scripts)
- **Conda** (Environment setup via environment.yml / environment-lock.yml before running the planner)

## Examples

```
conda activate AudioFab; python scripts/start_all.py
```

## Evaluation signals

- All subtasks are reported as successfully executed by the LLM Planner before the final response is generated
- Each expected artifact (accompaniment, vocal, style description, generated music) passes a file_exists check in RESULT_FOLDER_PATH
- scripts/check.sh validates that servers_config. points each service's command to a valid local Python interpreter
- Selected toolchain matches subtask semantics (e.g., Music Separation via Demucs for accompaniment extraction, Music Style Description via Qwen2-Audio, Music Generation via DiffRhythm)
- No tool hallucination: the invoked tool list is the filtered/shortened list with few-shot exemplars injected, not the entire tool library

## Limitations

- Local deployment of third-party models consumes substantial computing resources; the README recommends starting with API-based models via MD_TOKEN and HF_TOKEN
- If local deployment is incomplete, FunTTS_mcp_servers, music_mcp_servers, and Audioseparator_mcp_servers must be removed from servers_config., so those tools cannot be selected
- The official paper link is pending in the revised version and no changelog is available, limiting traceability of tool-selection updates

## Evidence
