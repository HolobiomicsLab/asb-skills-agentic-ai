---
name: privacy-pathway-internal-external-labeling
description: Use when auditing multi-agent LLM executions whose privacy-relevant data
  flows must be labeled by pathway (C1-C7) and classified internal vs. external, e.g.
  when output-only audits would miss C2/C5 violations.
when_to_use_negative:
- When the analysis targets only final-output (C1) leakage with no internal-channel
  instrumentation — output-only audits are exactly what this taxonomy is designed
  to extend beyond
- When the system being audited is single-agent and emits no inter-agent messages
  (C2) or shared memory writes (C5), making the internal/external split moot for the
  unobserved pathways
edam_operation: http://edamontology.org/operation_2990
tools:
- name: agentleak
  role: Core SDK whose checker module hosts the C1-C7 channel taxonomy and whose check()
    API receives the channel identifier
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Entry point whose check() method accepts the channel parameter (e.g. channel='C1')
    that the classification function must resolve to Internal or External
  repo: https://github.com/Privatris/AgentLeak
- name: pytest
  role: Runs the unit tests covering all seven channel identifiers under tests/ (pytest
    tests/ -v)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:4f06fa58c5874524785e7f8363c4bc00bae96c2254ba5e5ca6d6ec5ec50bef7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Privatris/AgentLeak
  tools:
  - AgentLeakTester
  - agentleak
  - pytest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# privacy-pathway-internal-external-labeling

## Summary

Implements AgentLeak's seven-channel privacy taxonomy (C1-C7) as a lookup that tags every captured multi-agent communication pathway as Internal or External, so leakage events from inter-agent messages (C2) and shared memory (C5) are distinguishable from output-only (C1) exposure.

## When to use

Use when auditing multi-agent LLM executions whose privacy-relevant data flows must be labeled by pathway (C1-C7) and classified internal vs. external, e.g. when output-only audits would miss C2/C5 violations.

## When NOT to use

- When the analysis targets only final-output (C1) leakage with no internal-channel instrumentation — output-only audits are exactly what this taxonomy is designed to extend beyond
- When the system being audited is single-agent and emits no inter-agent messages (C2) or shared memory writes (C5), making the internal/external split moot for the unobserved pathways

## Inputs

- Channel identifier string (e.g. 'C1'...'C7') as passed to AgentLeakTester.check
- Captured pathway events from multi-agent LLM executions (final outputs, inter-agent messages, tool inputs/outputs, memory writes, logs, artifacts)

## Outputs

- channels.tsv — three-column Channel/Type/Description table matching the methods specification
- Internal/External type label per channel event (2 Internal: C2, C5; 5 External: C1, C3, C4, C6, C7)

## How to apply

Instantiate the channel taxonomy in the checker module as an enumerated set of the seven pathways C1-C7 with their descriptions (C1 final output, C2 inter-agent messages, C3 tool inputs, C4 tool outputs, C5 memory writes, C6 logs, C7 artifacts), mirroring the methods Channels table. Assign each channel its documented type label: C2 and C5 as Internal; C1, C3, C4, C6, and C7 as External. Implement a classification function that accepts the same channel identifier passed to AgentLeakTester.check (e.g. channel='C1') and returns the internal/external label so every captured pathway event can be labeled. Emit the taxonomy as a three-column Channel/Type/Description table (channels.tsv) matching the methods specification, and add unit tests covering all seven identifiers under tests/ before running pytest tests/ -v.

## Related tools

- **agentleak** (Core SDK whose checker module hosts the C1-C7 channel taxonomy and whose check() API receives the channel identifier) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Entry point whose check() method accepts the channel parameter (e.g. channel='C1') that the classification function must resolve to Internal or External) — https://github.com/Privatris/AgentLeak
- **pytest** (Runs the unit tests covering all seven channel identifiers under tests/ (pytest tests/ -v))

## Examples

```
from agentleak import AgentLeakTester, DetectionMode; tester = AgentLeakTester(mode=DetectionMode.HYBRID); result = tester.check(vault={"ssn": "123-45-6789"}, output="The SSN is 123-45-6789", channel="C1")
```

## Evaluation signals

- Every identifier C1-C7 resolves to its documented type (2 Internal: C2, C5; 5 External: C1, C3, C4, C6, C7)
- No unmatched channel identifiers are returned by the classification function
- channels.tsv contains exactly seven rows with three columns (Channel/Type/Description) matching the methods table
- pytest tests/ -v passes fully with unit tests for all seven identifiers
- Labeled events can reproduce the aggregated internal-exposure metric across C1, C2, C5 reported in the benchmark

## Limitations

- Benchmark evaluation is focused on final outputs (C1), inter-agent messages (C2), and shared memory (C5); the advanced C3/C6 tool-and-log channels require a separate benchmark script (benchmark_tools.py)
- Internal-channel labels reflect AgentLeak's documented taxonomy; the aggregate exposure figure (68.9%) covers only C1, C2, C5, so external channel types do not feed that headline metric
- README Key Results table reports per-model C1/C2 figures and a per-model 'H1 (Audit Gap)' that differs from the article's 41.7% aggregate miss rate, so label consumers should not conflate the two

## Evidence

- [methods] Assign each channel its documented type label: C2 and C5 as Internal; C1, C3, C4, C6, and C7 as External.: [evidence span withheld — anchor status: not_found]
- [methods] Channels table with seven pathways: [evidence span withheld — anchor status: not_found]
- [readme] README scope lists the seven channels: [evidence span withheld — anchor status: external_doc]
- [readme] README usage shows the channel parameter on check(): "result = tester.check(
    vault={"ssn": "123-45-6789"},
    output="The SSN is 123-45-6789",
    channel="C1"
)"
