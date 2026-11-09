---
name: llm-subtask-decomposition-planning
description: Use when one natural-language audio request (e.g. emotion analysis plus
  speech generation plus digital-human video synthesis) must be split into ordered
  subtasks an agent can dispatch to external tools sequentially.
when_to_use_negative:
- 'The request is outside the audio domain: AudioFab is specifically designed for
  audio processing, and its nine MCP services cover only speech, music, and audio
  tools.'
- A needed subtask depends on models requiring local deployment whose services (FunTTS_mcp_servers,
  music_mcp_servers, Audioseparator_mcp_servers) were removed from servers_config.
  because local deployment was not completed.
- The goal is discovering or choosing a tool from an existing inventory rather than
  decomposing a command — that stage belongs to Tool Selection's listing, querying,
  and intelligent search.
tools:
- name: AudioFab
  role: Open-source agent framework whose fixed tool-learning control loop hosts the
    planning stage between task decomposition and response generation
  repo: https://github.com/SmileHnu/AudioFab
- name: LLM Planner
  role: Performs task decomposition and planning based on user commands, directs MCP
    Server tool invocations, and assembles the final response after all subtasks succeed
  repo: https://github.com/SmileHnu/AudioFab
- name: MCP Client
  role: Receives user commands, forwards them to the LLM Planner, and passes tool
    execution results back so the planner can make subsequent decisions
  repo: https://github.com/SmileHnu/AudioFab
- name: Python
  role: Runtime used to launch the fixed control loop via scripts/start_all.py and
    to execute each tool in its dedicated dependency environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:653b107962f9022f4afc31c9e5b8952901beb50c2a4dc055daf487847d6178cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - LLM Planner
  - MCP Client
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# llm-subtask-decomposition-planning

## Summary

The Task Planning stage of AudioFab's fixed tool-learning control loop: an LLM Planner (configured via LLM_API_KEY/LLM_BASE_URL/LLM_MODEL_NAME in .env) decomposes a natural-language user command into ordered subtasks, then iterates tool invocation and execution feedback through the MCP Client/Server until every subtask succeeds.

## When to use

Use when one natural-language audio request (e.g. emotion analysis plus speech generation plus digital-human video synthesis) must be split into ordered subtasks an agent can dispatch to external tools sequentially.

## When NOT to use

- The request is outside the audio domain: AudioFab is specifically designed for audio processing, and its nine MCP services cover only speech, music, and audio tools.
- A needed subtask depends on models requiring local deployment whose services (FunTTS_mcp_servers, music_mcp_servers, Audioseparator_mcp_servers) were removed from servers_config. because local deployment was not completed.
- The goal is discovering or choosing a tool from an existing inventory rather than decomposing a command — that stage belongs to Tool Selection's listing, querying, and intelligent search.

## Inputs

- Natural-language user command describing a multi-step audio task (e.g. separate vocals, analyze the accompaniment's musical style, create new music in that style)
- Uploaded audio files or file paths (.md, ., .txt) pointed to by MARKDOWN_FOLDER_PATH and RESULT_FOLDER_PATH in .env
- LLM endpoint settings in .env: LLM_API_KEY, LLM_BASE_URL, LLM_MODEL_NAME (or OLLAMA_MODEL_NAME/OLLAMA_BASE_URL)

## Outputs

- Ordered subtask plan derived from the user command
- Per-subtask tool invocation directives sent through the MCP Client to the MCP Server
- Single complete natural-language response delivered to the user only after every subtask succeeds

## How to apply

Configure the planner first: fill LLM_API_KEY, LLM_BASE_URL, and LLM_MODEL_NAME in the .env file (OLLAMA_MODEL_NAME/OLLAMA_BASE_URL for local Ollama models) and validate with scripts/check.sh. When the MCP Client receives a user command it forwards it to the LLM Planner, which performs task decomposition and planning into ordered subtasks. For each subtask the planner directs the MCP Server to invoke a tool and evaluates the toolchains returned by Tool Selection rather than reading the entire tool library, which the paper flags as a cause of excessive context and tool hallucination. Close the feedback loop: the MCP Client passes each tool's execution results back to the planner, which makes subsequent decisions before the next dispatch. Only after all subtasks report success does the LLM Planner assemble the complete response, which the MCP Client delivers to the user. Validate end-to-end by running python scripts/start_all.py and confirming one command traverses planning → tool selection → tool call/execution → response generation.

## Related tools

- **AudioFab** (Open-source agent framework whose fixed tool-learning control loop hosts the planning stage between task decomposition and response generation) — https://github.com/SmileHnu/AudioFab
- **LLM Planner** (Performs task decomposition and planning based on user commands, directs MCP Server tool invocations, and assembles the final response after all subtasks succeed) — https://github.com/SmileHnu/AudioFab
- **MCP Client** (Receives user commands, forwards them to the LLM Planner, and passes tool execution results back so the planner can make subsequent decisions) — https://github.com/SmileHnu/AudioFab
- **Python** (Runtime used to launch the fixed control loop via scripts/start_all.py and to execute each tool in its dedicated dependency environment)

## Examples

```
conda activate AudioFab && python scripts/start_all.py
```

## Evaluation signals

- scripts/check.sh passes after .env and mcp_servers/servers_config. are edited (each service's 'command' key points at a resolvable Python interpreter path).
- One natural-language command issued after `python scripts/start_all.py` completes planning → tool selection → tool call/execution → response generation end-to-end.
- No final response is emitted while any subtask is pending; the complete response appears only after every subtask reports success.
- Tool execution results are returned to the LLM Planner for subsequent decisions, showing an iterated loop rather than one-shot dispatch.
- The planner consumes toolchains provided by Tool Selection (intelligent-filtered list plus few-shot exemplars), not the full tool inventory.

## Limitations

- Locally deploying the FunTTS, Music, and Audio Separator models consumes substantial computing resources; if those services are unregistered, AudioFab will not use models requiring local deployment, so planner subtasks cannot target them.
- Burying the planner in the entire tool library causes excessive contextual information and tool hallucination; planning must rely on the two-stage filtered toolchain instead.
- The paper link is not yet available in the official revised version (ACM MM acceptance), so method details lean on the repository README.

## Evidence

- [intro] By coordinating key stages of tool learning, LLMs can use external tools to complete complex tasks, leading to the creation of LLM-based agents (e.g., METAGPT [4]): [evidence span withheld — anchor status: not_found]
