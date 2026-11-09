---
name: llm-benchmark-channel-instrumentation
description: Use when building or auditing a multi-agent LLM privacy benchmark and
  you need every capture pathway (final output, inter-agent messages, tool inputs/outputs,
  memory writes, logs, artifacts) labeled and leak-checked per channel.
when_to_use_negative:
- Input is a single-agent system where only the final output channel exists — the
  C2/C5 internal channels are meaningless without inter-agent coordination
- You need to audit compliance of a deployed system's published outputs only, not
  instrument internal execution pathways
- The framework is not built on CrewAI/LangGraph-style agent orchestration, so tool,
  memory, and log captures (C3-C6) cannot be intercepted
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: agentleak
  role: Core benchmark SDK installed via 'pip install -e .' providing the channel-aware
    leak testing framework
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Checker class that accepts a channel identifier (e.g. channel='C1') plus vault
    and output, returning leaked/confidence results per channel
  repo: https://github.com/Privatris/AgentLeak
- name: pytest
  role: Runs the unit tests covering all seven channel identifiers ('pytest tests/
    -v')
  repo: https://github.com/Privatris/AgentLeak
- name: CrewAI
  role: Agent framework whose crews can be wrapped via add_agentleak_to_crew to capture
    internal-channel events
  repo: https://github.com/Privatris/AgentLeak
- name: LangGraph
  role: Agent framework whose graphs can be wrapped via add_agentleak_to_langgraph
    to capture internal-channel events
  repo: https://github.com/Privatris/AgentLeak
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:429dd823c0ce80efd91308178a35651b83c99b17ec71f4bd1c48acaef3f1a8d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Privatris/AgentLeak
  tools:
  - AgentLeakTester
  - CrewAI
  - LangGraph
  - agentleak
  - pytest
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# llm-benchmark-channel-instrumentation

## Summary

Instruments the seven privacy-relevant communication channels (C1-C7) of a multi-agent LLM benchmark so that leak checks cover inter-agent messages, memory, tools, logs, and artifacts rather than final outputs alone, since output-only audits miss a large fraction of violations.

## When to use

Use when building or auditing a multi-agent LLM privacy benchmark and you need every capture pathway (final output, inter-agent messages, tool inputs/outputs, memory writes, logs, artifacts) labeled and leak-checked per channel.

## When NOT to use

- Input is a single-agent system where only the final output channel exists — the C2/C5 internal channels are meaningless without inter-agent coordination
- You need to audit compliance of a deployed system's published outputs only, not instrument internal execution pathways
- The framework is not built on CrewAI/LangGraph-style agent orchestration, so tool, memory, and log captures (C3-C6) cannot be intercepted

## Inputs

- Secret vault (e.g. {"ssn": "123-45-6789"}) with known sensitive values
- Candidate outputs/messages captured from agent execution on channels C1-C7 (strings)
- Channel identifier strings 'C1'..'C7' passed to AgentLeakTester.check
- DetectionMode setting (HYBRID for production) for the AgentLeakTester instance

## Outputs

- channels.tsv: three-column Channel/Type/Description table matching the methods specification
- Per-check result objects with leaked flag and confidence, attributed to a channel identifier
- Internal/External type label for every C1-C7 identifier
- pytest test suite covering all seven channel identifiers

## How to apply

Instantiate the channel taxonomy as an enumerated set of seven pathways in the checker module — C1 final output, C2 inter-agent messages, C3 tool inputs, C4 tool outputs, C5 memory writes, C6 logs, C7 artifacts — mirroring the methods Channels table. Assign each channel its documented type label (C2 and C5 Internal; C1, C3, C4, C6, C7 External) and implement a classification function that accepts the same channel identifier passed to AgentLeakTester.check (e.g. channel='C1') and returns the internal/external label. Configure the tester with DetectionMode.HYBRID (~200ms latency, designated for production) and pass a secret vault plus candidate output per check so leaked/confident verdicts can be attributed to a specific channel. Emit the taxonomy as a three-column Channel/Type/Description table (channels.tsv) and add unit tests covering all seven identifiers. Rationale: AgentLeak shows C2 leaks at 68.8% versus 27.2% for C1, so restricting instrumentation to C1 hides internal-channel exposure (total 68.9% across C1+C2+C5).

## Related tools

- **agentleak** (Core benchmark SDK installed via 'pip install -e .' providing the channel-aware leak testing framework) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Checker class that accepts a channel identifier (e.g. channel='C1') plus vault and output, returning leaked/confidence results per channel) — https://github.com/Privatris/AgentLeak
- **pytest** (Runs the unit tests covering all seven channel identifiers ('pytest tests/ -v')) — https://github.com/Privatris/AgentLeak
- **CrewAI** (Agent framework whose crews can be wrapped via add_agentleak_to_crew to capture internal-channel events) — https://github.com/Privatris/AgentLeak
- **LangGraph** (Agent framework whose graphs can be wrapped via add_agentleak_to_langgraph to capture internal-channel events) — https://github.com/Privatris/AgentLeak

## Examples

```
from agentleak import AgentLeakTester, DetectionMode; tester = AgentLeakTester(mode=DetectionMode.HYBRID); result = tester.check(vault={"ssn": "123-45-6789"}, output="The SSN is 123-45-6789", channel="C1")
```

## Evaluation signals

- Every C1-C7 identifier resolves to its documented type with no unmatched channel identifiers (2 internal: C2, C5; 5 external: C1, C3, C4, C6, C7)
- channels.tsv has exactly three columns (Channel/Type/Description) and seven rows matching the methods table
- pytest tests/ -v passes fully, including unit tests for all seven identifiers
- tester.check on a vaulted SSN echoed in the output with channel='C1' returns result.leaked=True with nonzero confidence
- Aggregated audit-gap metric (C2 leak rate minus C1 leak rate) is computable; a non-negative gap is expected since C2 >= C1 holds across all five models

## Limitations

- Audit-gap magnitude is model-dependent: Claude 3.5 Sonnet shows the lowest C1 leakage (8.2%) but the highest internal/external ratio (6.6x), so C1-only baselines underestimate exposure unevenly across models
- Channel-level results differ per benchmark script: main results cover C1/C2/C5 while C3/C6 require a separate tools-and-logs benchmark run
- Detection in HYBRID mode adds ~200ms latency per check, which may constrain very large trace counts

## Evidence

- [methods] AgentLeakTester.check with channel identifier: "from agentleak import AgentLeakTester, DetectionMode"
- [abstract] C2 vs C1 audit gap: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [readme] seven-channel scope: [evidence span withheld — anchor status: external_doc]
- [readme] tool/log secondary-channel leakage: [evidence span withheld — anchor status: external_doc]
- [methods] HYBRID mode designation: "HYBRID | ~200ms | Production"
