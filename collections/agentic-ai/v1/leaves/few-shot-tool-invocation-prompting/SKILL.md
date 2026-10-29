---
name: few-shot-tool-invocation-prompting
description: Use when an LLM agent must choose tools from a large registered inventory
  (e.g. a tool JSON catalog) and is prone to tool hallucination or erroneous calls
  caused by excessive contextual information.
when_to_use_negative:
- The tool inventory is tiny enough to fit fully in context without filtering — the
  two-stage strategy targets excessive contextual information from entire tool libraries
- The agent does not perform tool selection at all (e.g. a fixed single-tool pipeline),
  where filtering and exemplars add overhead without benefit
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3071
tools:
- name: AudioFab
  role: Agent framework implementing the two-stage intelligent selection workflow
    (filtering then few-shot injection) inside its Tool Selection stage
  repo: https://github.com/SmileHnu/AudioFab
- name: Tool Query Servers
  role: MCP service providing listing, querying, and intelligent search over the tool
    JSON to support tool discovery and information queries
  repo: https://github.com/SmileHnu/AudioFab
- name: LLM Planner
  role: Consumes the filtered tool list and few-shot exemplars, evaluates toolchains,
    and directs tool invocation
  repo: https://github.com/SmileHnu/AudioFab
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:deb9e097751611c2167fad2951748d216b9c9ee2e01b9d551439bd3e4cbbde96
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - LLM Planner
  - Python
  - Tool Query Servers
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# few-shot-tool-invocation-prompting

## Summary

AudioFab's two-stage anti-hallucination strategy for tool selection: first intelligently filter the tool list to shorten LLM context, then inject few-shot exemplars into the prompt before tool invocation so the planner picks the correct tool and parameters.

## When to use

Use when an LLM agent must choose tools from a large registered inventory (e.g. a tool JSON catalog) and is prone to tool hallucination or erroneous calls caused by excessive contextual information.

## When NOT to use

- The tool inventory is tiny enough to fit fully in context without filtering — the two-stage strategy targets excessive contextual information from entire tool libraries
- The agent does not perform tool selection at all (e.g. a fixed single-tool pipeline), where filtering and exemplars add overhead without benefit

## Inputs

- Tool inventory JSON file containing tool names, descriptions, and examples
- Decomposed subtasks from the LLM Planner's task planning stage
- Registered MCP tool list from ToolKits.md / mcp_servers/servers_config.json

## Outputs

- Filtered shortlist of candidate tools for the current subtask
- Few-shot exemplar-augmented prompt for tool invocation
- Selected tool and parameters forwarded to the MCP Client for execution

## How to apply

Enumerate registered tools from the tool inventory JSON (tool names, descriptions, examples) served by the Tool Query Servers, which expose listing, querying, and intelligent search over the catalog. Apply stage one, intelligent filtering: streamline the full tool list down to a short candidate set relevant to the current subtask, because existing systems that burden the LLM with entire tool libraries risk interpretive errors ('tool hallucination'). Apply stage two, few-shot exemplar injection: before invocation, prepend worked exemplars for the candidate tools so the LLM Planner receives precise guidance on which tool to call and how. Feed the filtered, exemplar-augmented toolchain to the LLM Planner for evaluation, then let the MCP Client send the invocation request to the MCP Server in the tool's dedicated dependency environment. Rationale: shortening context improves efficiency while exemplars improve accuracy in complex audio tasks.

## Related tools

- **AudioFab** (Agent framework implementing the two-stage intelligent selection workflow (filtering then few-shot injection) inside its Tool Selection stage) — https://github.com/SmileHnu/AudioFab
- **Tool Query Servers** (MCP service providing listing, querying, and intelligent search over the tool JSON to support tool discovery and information queries) — https://github.com/SmileHnu/AudioFab
- **LLM Planner** (Consumes the filtered tool list and few-shot exemplars, evaluates toolchains, and directs tool invocation) — https://github.com/SmileHnu/AudioFab

## Examples

```
python scripts/start_all.py
```

## Evaluation signals

- The filtered tool list is a strict subset of the full registered inventory and noticeably shorter, shortening the prompt context
- Few-shot exemplars appear in the prompt before any invocation is issued
- The selected tool matches the exemplar-guided choice and invocation succeeds in the tool's dedicated dependency environment
- No tool hallucination: every call resolves to a registered tool with valid parameters from the tool JSON
- End-to-end command completes planning -> tool selection -> tool call/execution -> response generation after launching python scripts/start_all.py

## Limitations

- Intelligent filtering may omit a needed tool if the retrieval/ranking is imperfect; the dev1.1 branch updated the retrieval model and retrieval ranking algorithm
- Exemplars must be kept in sync with the registered tool inventory in ToolKits.md, or guidance becomes stale
- The strategy suppresses but does not guarantee elimination of erroneous model calls; stability relies on both stages being applied

## Evidence

- [intro] hallucination cause: [evidence span withheld — anchor status: not_found]
- [methods] tool query servers: [evidence span withheld — anchor status: not_found]
