---
name: audio-tool-retrieval-matching
description: Use when an LLM agent must select the most suitable audio tool from a
  large registered inventory and needs to suppress tool hallucination by filtering
  the tool list and injecting few-shot exemplars before invocation.
when_to_use_negative:
- There is only a single candidate tool for the subtask, so filtering/retrieval adds
  latency without value
- The tool inventory has not been registered — e.g., FunTTS, Music, and Audio Separator
  services were removed from servers_config.json because local models are not deployed
- The task is pure audio signal processing with a fixed known tool, not open-ended
  multi-tool orchestration
edam_operation: http://edamontology.org/operation_0306
tools:
- name: AudioFab
  role: Host framework whose fixed tool-learning control loop provides the Tool Selection
    Module (enumeration, retrieval matching, parameter querying)
  repo: https://github.com/SmileHnu/AudioFab
- name: MCP Server (Tool Query Servers)
  role: Provides the Tool Selection service via listing, querying, and intelligent
    search over the tool JSON for tool discovery and information queries
  repo: https://github.com/SmileHnu/AudioFab
- name: Python
  role: Runtime for the MCP Server scripts and interpreter path declared in mcp_servers/servers_config.json
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:950349112243ec299ba2a30759e2ba76cc8ef5a419bd2d5c7c46b9476bee63bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - MCP Server (Tool Query Servers)
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# audio-tool-retrieval-matching

## Summary

Retrieval-based tool matching from AudioFab: enumerate registered tools from a JSON inventory, run retrieval matching via Tool Query Servers (listing, querying, intelligent search), and apply a two-stage anti-hallucination strategy (intelligent filtering + few-shot exemplar injection) before invocation.

## When to use

Use when an LLM agent must select the most suitable audio tool from a large registered inventory and needs to suppress tool hallucination by filtering the tool list and injecting few-shot exemplars before invocation.

## When NOT to use

- There is only a single candidate tool for the subtask, so filtering/retrieval adds latency without value
- The tool inventory has not been registered — e.g., FunTTS, Music, and Audio Separator services were removed from servers_config.json because local models are not deployed
- The task is pure audio signal processing with a fixed known tool, not open-ended multi-tool orchestration

## Inputs

- JSON tool inventory (tool names, descriptions, examples) documented in ToolKits.md
- User natural-language command decomposed into ordered subtasks by the LLM Planner
- Registered MCP Server services in mcp_servers/servers_config.json

## Outputs

- A ranked/selected tool (or toolchain) per subtask from the Tool Selection service
- Shortened context (filtered tool list) plus few-shot exemplars injected into the LLM prompt
- Tool invocation requests forwarded from the MCP Client to the MCP Server

## How to apply

Load the tool inventory as a JSON file containing tool names, descriptions, and examples, and enumerate the tools registered under the MCP Server services (e.g., DSP Servers, Audio Servers, FunTTS/Music/Audio Separator MCP Servers). Apply the two-stage anti-hallucination strategy: first, intelligent filtering that streamlines the tool list to shorten the LLM context, since burdening the LLM with the entire tool library causes excessive contextual information and tool hallucination; second, few-shot exemplar injection for precise guidance before invocation. Use the Tool Query Servers, which offer listing, querying, and intelligent search over the tool JSON, to match candidate tools to the subtask produced by the LLM Planner's task decomposition. The LLM Planner then evaluates the toolchains returned by Tool Selection and directs the MCP Client to send invocation requests; after each tool executes in its dedicated dependency environment, results return to the planner for the next decision. Rationale: coordinating task decomposition, tool selection, and execution feedback is what the paper identifies as the missing collaborative mechanism in current audio agents.

## Related tools

- **AudioFab** (Host framework whose fixed tool-learning control loop provides the Tool Selection Module (enumeration, retrieval matching, parameter querying)) — https://github.com/SmileHnu/AudioFab
- **MCP Server (Tool Query Servers)** (Provides the Tool Selection service via listing, querying, and intelligent search over the tool JSON for tool discovery and information queries) — https://github.com/SmileHnu/AudioFab
- **Python** (Runtime for the MCP Server scripts and interpreter path declared in mcp_servers/servers_config.json)

## Examples

```
conda activate AudioFab && python scripts/start_all.py
```

## Evaluation signals

- The filtered tool list passed to the LLM is substantially shorter than the full registered inventory (context-shortening check)
- Selected tool's name, description, and examples all appear in the tool JSON inventory — no invented (hallucinated) tool names
- One natural-language command completes planning -> tool selection -> tool call/execution -> response generation end-to-end
- Few-shot exemplars are present in the prompt before invocation and the final response is emitted only after every subtask reports success
- scripts/check.sh reports the servers_config.json configuration is correct before the loop is launched

## Limitations

- Retrieval model and ranking algorithm changed in the dev1.1 branch, so matching behavior differs across branches
- Selection only covers tools registered in servers_config.json; removing FunTTS/Music/Audio Separator services silently shrinks the candidate pool
- Existing systems that burden the LLM with entire tool libraries risk excessive contextual information and tool hallucination — the fix depends on good few-shot exemplars

## Evidence

- [methods] first, it streamlines the tool list via intelligent filtering to shorten the context: "first, it streamlines the tool list via intelligent filtering to shorten the context"
- [methods] Offers services for tool discovery and tool information queries.: [evidence span withheld — anchor status: not_found]
- [readme] AudioFab addresses tool hallucination through a two-stage strategy: "AudioFab addresses tool hallucination through a two-stage strategy: first, it streamlines the tool list via intelligent filtering to shorten the context; second, it injects few-shot exemplars for"
