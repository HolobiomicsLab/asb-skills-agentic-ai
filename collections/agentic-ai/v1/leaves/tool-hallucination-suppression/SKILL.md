---
name: tool-hallucination-suppression
description: Use when an LLM agent must select tools from a large registered tool
  inventory (e.g., a JSON tool file) and risks 'tool hallucination' — interpretive
  errors from burdening the model with the entire tool library.
when_to_use_negative:
- The tool inventory is tiny (a handful of tools) — filtering overhead is unnecessary
  and the whole list fits in context safely
- The agent architecture does not route selection through a queryable tool registry
  (no JSON tool file or listing/querying service), so there is nothing to filter or
  exemplify
- You need free-form model behavior rather than a fixed tool-learning control loop
  — AudioFab's fixed loop and exemplar injection constrain which tools can be invoked
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3071
tools:
- name: AudioFab
  role: Agent framework implementing the two-stage intelligent-selection workflow
    over its MCP-based tool inventory
  repo: https://github.com/SmileHnu/AudioFab
- name: Python
  role: Runtime for the LLM Planner, MCP Client, and MCP Server services that execute
    filtering and exemplar injection
- name: LangChain
  role: Compared baseline lacking the anti-hallucination two-stage strategy (AudioGPT/LangChain
    marked with tool-hallucination failure columns in the intro table)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1145/3746027.3756869
    title: 'AudioFab: Building A General and Intelligent Audio Factory through Tool
      Learning'
  source_content_hash: sha256:5da1e1b3d313975e61494b9eeb77fc465c361cce9a8e4edae786083ec2874059
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SmileHnu/AudioFab
  tools:
  - AudioFab
  - LangChain
  - Python
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tool-hallucination-suppression

## Summary

A two-stage anti-hallucination strategy from AudioFab that suppresses erroneous LLM tool calls: intelligent filtering first streamlines the tool list to shorten the context, then few-shot exemplars are injected for precise guidance before invocation.

## When to use

Use when an LLM agent must select tools from a large registered tool inventory (e.g., a JSON tool file) and risks 'tool hallucination' — interpretive errors from burdening the model with the entire tool library.

## When NOT to use

- The tool inventory is tiny (a handful of tools) — filtering overhead is unnecessary and the whole list fits in context safely
- The agent architecture does not route selection through a queryable tool registry (no JSON tool file or listing/querying service), so there is nothing to filter or exemplify
- You need free-form model behavior rather than a fixed tool-learning control loop — AudioFab's fixed loop and exemplar injection constrain which tools can be invoked

## Inputs

- JSON tool inventory file with tool names, descriptions, and examples (documented in ToolKits.md)
- Ordered subtasks produced by the LLM Planner from a natural-language user command
- MCP Server tool listing/querying/intelligent-search results from Tool Query Servers

## Outputs

- A filtered, shortened tool list per decision step
- Few-shot exemplars injected into the planner prompt before invocation
- A validated tool selection / toolchain for each subtask, forwarded to the MCP Server for execution

## How to apply

Maintain the tool inventory in a JSON file of tool names, descriptions, and examples, and expose it through Tool Query Servers offering listing, querying, and intelligent search services. Before each invocation, run stage one: intelligent filtering that streamlines the full tool list down to a short context, reducing excessive contextual information. Then run stage two: inject few-shot exemplars into the prompt so the LLM Planner receives precise guidance for the exact call. The LLM Planner evaluates the toolchains provided by Tool Selection, the MCP Client sends invocation requests to the MCP Server, and results return to the planner for the next decision. Rationale: existing audio agents burden LLMs with whole tool libraries, which increases contextual load and the risk of tool hallucination; filtering plus exemplar injection keeps the context short while anchoring selection to demonstrated correct usage patterns.

## Related tools

- **AudioFab** (Agent framework implementing the two-stage intelligent-selection workflow over its MCP-based tool inventory) — https://github.com/SmileHnu/AudioFab
- **Python** (Runtime for the LLM Planner, MCP Client, and MCP Server services that execute filtering and exemplar injection)
- **LangChain** (Compared baseline lacking the anti-hallucination two-stage strategy (AudioGPT/LangChain marked with tool-hallucination failure columns in the intro table))

## Examples

```
conda activate AudioFab; python scripts/start_all.py  # then submit one natural-language command and verify planning -> selection -> execution -> response
```

## Evaluation signals

- No erroneous tool calls: the agent invokes only tools present in the filtered candidate list, and every selected tool matches its few-shot exemplar pattern
- Context shortening is observed: the planner prompt contains a streamlined tool list rather than the full ToolKits.md inventory
- End-to-end loop succeeds: one natural-language command completes planning -> tool selection -> tool call/execution -> response generation, with the final response emitted only after every subtask reports success
- Tool Query Servers return correct candidates via listing, querying, and intelligent search over the tool JSON
- Registry integrity: new tools integrate through the simple registration process and become selectable without dependency conflicts

## Limitations

- Existing systems that burden LLMs with entire tool libraries suffer excessive contextual information and tool hallucination — the strategy only helps if filtering is actually applied before prompting
- Few-shot exemplar injection requires maintained exemplars in the tool JSON; poorly documented tools yield weak guidance
- Tools requiring local deployment (FunTTS, Music, Audio Separator MCP Servers) must be deregistered from servers_config. if not deployed, shrinking the selectable inventory

## Evidence

- [methods] two-stage anti-hallucination strategy: [evidence span withheld — anchor status: not_found]
- [intro] intro problem statement: [evidence span withheld — anchor status: not_found]
- [methods] Tool Query Servers: [evidence span withheld — anchor status: not_found]
