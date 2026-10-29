---
name: benchmark-metric-recomputation
description: Use when trace-level benchmark outputs must be re-aggregated into published
  channel-level metrics and validated against reported figures, e.g. reproducing 68.9%
  total exposure or a 41.7% audit gap from C1/C2/C5 leakage rates.
when_to_use_negative:
- When the analysis targets tool-input (C3) or system-log (C6) secondary channels
  — use the benchmark_tools.py pipeline instead of the main C1/C2/C5 benchmark
- When only final-output (C1) auditing is in scope and internal-channel leakage is
  explicitly out of scope — the union-based exposure computation would overstate the
  metric of interest
- 'When traces were produced with a different benchmark version whose per-model rates
  (README table: C1 28.2%, C2 74.0%, total 79.7%) do not correspond to the paper''s
  abstract figures'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: agentleak
  role: Core benchmark SDK installed via 'pip install -e .'; provides the trace generation
    CLI and scoring framework
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Scores each trace's channel output against the vault using 'from agentleak
    import AgentLeakTester, DetectionMode' with DetectionMode.HYBRID (~200ms latency)
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o-mini
  role: Model under test for the reproduction command 'python benchmark.py --n 100
    --traces --model openai/gpt-4o-mini'
- name: CrewAI
  role: Multi-agent framework used to generate the inter-agent (C2) message traces
    being scored
- name: LangGraph
  role: Alternative multi-agent framework ('from langgraph.graph import StateGraph')
    for generating traces with internal channels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:aa330d9aaad8acea73c4c61e09b7b61e2465d474e85303cecc4596df33ec8f01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Privatris/AgentLeak
  tools:
  - AgentLeakTester
  - CrewAI
  - GPT-4o
  - GPT-4o-mini
  - LangGraph
  - agentleak
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# benchmark-metric-recomputation

## Summary

Recompute headline benchmark metrics (total system exposure across channels C1/C2/C5 and the output-only audit-miss fraction) from raw multi-agent trace data using the AgentLeak package, validating against reported values (68.9% total exposure; 41.7% audit miss).

## When to use

Use when trace-level benchmark outputs must be re-aggregated into published channel-level metrics and validated against reported figures, e.g. reproducing 68.9% total exposure or a 41.7% audit gap from C1/C2/C5 leakage rates.

## When NOT to use

- When the analysis targets tool-input (C3) or system-log (C6) secondary channels — use the benchmark_tools.py pipeline instead of the main C1/C2/C5 benchmark
- When only final-output (C1) auditing is in scope and internal-channel leakage is explicitly out of scope — the union-based exposure computation would overstate the metric of interest
- When traces were produced with a different benchmark version whose per-model rates (README table: C1 28.2%, C2 74.0%, total 79.7%) do not correspond to the paper's abstract figures

## Inputs

- Multi-agent benchmark traces in benchmarks/ieee_repro/results/traces/ (generated via 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini')
- Vault of sensitive values per scenario (e.g. {'ssn': '123-45-6789'}) used by AgentLeakTester.check
- Channel labels C1 (external output), C2 (inter-agent messages), C5 (memory writes)

## Outputs

- channel_exposure_aggregation.csv with per-channel leakage rates, total system exposure (68.9%), and audit-miss fraction (41.7%)
- Per-channel leakage rates (C1 = 27.2%, C2 = 68.8%) recomputed from traces

## How to apply

Run the benchmark to generate traces ('python benchmark.py --n 100 --traces --model openai/gpt-4o-mini') and load them from benchmarks/ieee_repro/results/traces/. Score each trace's channel outputs against the vault with AgentLeakTester in DetectionMode.HYBRID (~200ms latency, designated for production use), flagging leakage on C1 (external final output), C2 (inter-agent messages), and C5 (memory writes). Aggregate per-channel leakage rates with pandas, computing total system exposure as the union of violations across C1, C2, and C5. Recompute the audit-miss fraction as the excess of inter-agent (C2) leakage over final-output (C1) leakage: 68.8% - 27.2% = 41.7 percentage points of violations invisible to a C1-only audit. Write per-channel rates, total exposure, and audit-miss fraction to channel_exposure_aggregation.csv and check the recomputed values match the reported 68.9% and 41.7%.

## Related tools

- **agentleak** (Core benchmark SDK installed via 'pip install -e .'; provides the trace generation CLI and scoring framework) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Scores each trace's channel output against the vault using 'from agentleak import AgentLeakTester, DetectionMode' with DetectionMode.HYBRID (~200ms latency)) — https://github.com/Privatris/AgentLeak
- **GPT-4o-mini** (Model under test for the reproduction command 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini')
- **CrewAI** (Multi-agent framework used to generate the inter-agent (C2) message traces being scored)
- **LangGraph** (Alternative multi-agent framework ('from langgraph.graph import StateGraph') for generating traces with internal channels)

## Examples

```
cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini && python -c "from agentleak import AgentLeakTester, DetectionMode; t=AgentLeakTester(mode=DetectionMode.HYBRID); r=t.check(vault={'ssn':'123-45-6789'}, output='The SSN is 123-45-6789', channel='C1'); print(r.leaked)"
```

## Evaluation signals

- Recomputed total system exposure across C1∪C2∪C5 matches the reported 68.9%
- Recomputed audit-miss fraction equals C2 − C1 = 68.8% − 27.2% = 41.7 percentage points
- Per-channel rate C2 (68.8%) exceeds C1 (27.2%), consistent with the paper's stated invariant that C2 ≥ C1 holds across all five models and four domains
- channel_exposure_aggregation.csv contains per-channel rates, total exposure, and audit-miss fraction columns as specified in the workflow
- HYBRID-mode scoring completes with ~200ms per-check latency and no scoring errors across the trace batch

## Limitations

- Per-model values in the README (Average C1 28.2%, C2 74.0%, Total Leak 79.7%) differ from the abstract's aggregated figures (C1 27.2%, C2 68.8%, total 68.9%), so the validation targets depend on which aggregation basis is used
- The audit-miss fraction is derived arithmetically (C2 − C1), so any per-channel scoring noise propagates directly into the headline metric
- Detection latency of ~200ms per check in HYBRID mode makes full-scale scoring of all 1,000 scenarios × 5 models non-trivial; results are sensitive to n (README shows both --n 100 and --n 1000 runs)

## Evidence

- [abstract] Aggregated across the final-output (C1), inter-agent message (C2), and memory (C5) channels, total system exposure reaches 68.9%: [evidence span withheld — anchor status: not_found]
- [abstract] Inter-agent messages (C2) leak at 68.8%, versus 27.2% for final outputs (C1), so output-only audits miss 41.7% of violations: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [methods] Trace generation and scoring procedure grounded in the methods section: "from agentleak import AgentLeakTester, DetectionMode"
- [methods] HYBRID detection mode latency and intended deployment tier: "HYBRID | ~200ms | Production"
- [readme] Benchmark reproduction command and trace location from the README: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] Per-model results table with different aggregation basis than the abstract: [evidence span withheld — anchor status: not_found]
