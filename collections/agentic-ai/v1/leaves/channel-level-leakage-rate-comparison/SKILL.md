---
name: channel-level-leakage-rate-comparison
description: Use when reproducing AgentLeak results and you need to verify that inter-agent
  message leakage (C2) consistently meets or exceeds final-output leakage (C1) for
  every model-domain cell.
when_to_use_negative:
- When analyzing secondary channels such as tool inputs (C3) or system logs (C6) —
  these require the tools/logs benchmark (benchmark_tools.py) and are covered by Finding
  7, not the C1 vs C2 inequality
- When the goal is measuring total system exposure (aggregated across C1, C2, C5 to
  68.9%) rather than the pairwise C2 >= C1 cell-level comparison
- When no trace data per model-domain cell exists (e.g. a single aggregate run without
  --traces), since the 20-cell verification cannot be performed
tools:
- name: agentleak
  role: SDK and CLI used to run the benchmark scenarios and produce the trace files
    that the comparison aggregates
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Leakage detector run in DetectionMode.HYBRID on channels C1 and C2 to classify
    leaked vs non-leaked traces
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o
  role: One of the five production LLMs evaluated as a model cell in the comparison
- name: GPT-4o-mini
  role: Model used for the traced reproduction run benchmark.py --n 100 --traces --model
    openai/gpt-4o-mini
- name: Claude 3.5 Sonnet
  role: One of the five production LLMs evaluated as a model cell in the comparison
- name: Mistral Large
  role: One of the five production LLMs evaluated as a model cell in the comparison
- name: Llama 3.3 70B
  role: One of the five production LLMs evaluated as a model cell in the comparison
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:89fa6584512910f661a75a7b8595c9cbd0424d069a8b24e1b65fa4835ed8d5a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Privatris/AgentLeak
  tools:
  - AgentLeakTester
  - Claude 3.5 Sonnet
  - GPT-4o
  - GPT-4o-mini
  - Llama 3.3 70B
  - Mistral Large
  - agentleak
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# channel-level-leakage-rate-comparison

## Summary

A benchmark-verification skill that aggregates AgentLeak trace results to compute per-model, per-domain leakage rates for channel C1 (final output) and C2 (inter-agent messages) and evaluates the C2 >= C1 inequality across all 20 model-domain cells.

## When to use

Use when reproducing AgentLeak results and you need to verify that inter-agent message leakage (C2) consistently meets or exceeds final-output leakage (C1) for every model-domain cell.

## When NOT to use

- When analyzing secondary channels such as tool inputs (C3) or system logs (C6) — these require the tools/logs benchmark (benchmark_tools.py) and are covered by Finding 7, not the C1 vs C2 inequality
- When the goal is measuring total system exposure (aggregated across C1, C2, C5 to 68.9%) rather than the pairwise C2 >= C1 cell-level comparison
- When no trace data per model-domain cell exists (e.g. a single aggregate run without --traces), since the 20-cell verification cannot be performed

## Inputs

- AgentLeak benchmark trace files (JSON) in benchmarks/ieee_repro/results/traces/
- Per-channel leakage records from AgentLeakTester checks on channels C1 and C2, keyed by model (5 models) and domain (healthcare, finance, legal, corporate)
- leakage_rates.csv aggregating per-model C1 (single-agent and multi-agent) and C2 rates

## Outputs

- c2_geq_c1_verification.csv with columns model, domain, c1_rate, c2_rate, c2_geq_c1 (20 rows)
- Overall proportion of model-domain cells satisfying C2 >= C1 (expected: 1.0)
- Comparison table of headline rates: single-agent C1 43.2%, multi-agent C1 27.2%, C2 68.8%

## How to apply

Load the reproduced trace files from benchmarks/ieee_repro/results/traces/ (produced by 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini') and aggregate channel-level leakage counts with pandas to compute per-model, per-domain leakage rates for channel C1 (final output) and channel C2 (inter-agent messages) across the 5 models (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) and 4 domains (healthcare, finance, legal, corporate). For each of the 20 model-domain cells, evaluate the inequality C2_rate >= C1_rate and record the cell-level pass/fail outcome. Write the resulting table (model, domain, c1_rate, c2_rate, c2_geq_c1) to c2_geq_c1_verification.csv and report the overall proportion of cells satisfying the inequality. Leakage classification should use the paper's three-tier detection pipeline via AgentLeakTester in DetectionMode.HYBRID, which is the designated production mode. The paper's acceptance criterion is a 100% pass rate over the 20 cells; the reported headline rates to approximate are single-agent C1 at 43.2%, multi-agent C1 at 27.2%, and C2 at 68.8%.

## Related tools

- **agentleak** (SDK and CLI used to run the benchmark scenarios and produce the trace files that the comparison aggregates) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Leakage detector run in DetectionMode.HYBRID on channels C1 and C2 to classify leaked vs non-leaked traces) — https://github.com/Privatris/AgentLeak
- **GPT-4o** (One of the five production LLMs evaluated as a model cell in the comparison)
- **GPT-4o-mini** (Model used for the traced reproduction run benchmark.py --n 100 --traces --model openai/gpt-4o-mini)
- **Claude 3.5 Sonnet** (One of the five production LLMs evaluated as a model cell in the comparison)
- **Mistral Large** (One of the five production LLMs evaluated as a model cell in the comparison)
- **Llama 3.3 70B** (One of the five production LLMs evaluated as a model cell in the comparison)

## Examples

```
cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- c2_geq_c1_verification.csv contains exactly 20 rows (5 models x 4 domains) with columns model, domain, c1_rate, c2_rate, c2_geq_c1
- 100% of the 20 model-domain cells satisfy C2_rate >= C1_rate (paper acceptance criterion)
- Aggregated rates approximate the reported values: single-agent C1 ~43.2%, multi-agent C1 ~27.2%, C2 ~68.8%
- The computed audit gap (C2 rate minus multi-agent C1 rate) is consistent with the reported ~41.7% of violations missed by output-only audits
- Rates are computed on the same trace set with leakage classification by AgentLeakTester in DetectionMode.HYBRID, matching the production-designated detector

## Limitations

- The inequality is verified over the 5-model x 4-domain grid of the AgentLeak evaluation; new models or domains are outside the verified 20 cells
- Detection depends on the HYBRID three-tier pipeline (~200ms latency per check); single-agent C1 baselines and multi-agent C1 must come from matched scenario counts to be comparable
- Output-only audits inherently miss inter-agent violations (~41.7% of violations), so conclusions drawn from C1 alone will understate system exposure

## Evidence

- [abstract] Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [abstract] Across all five models and four domains, the pattern C2 ≥ C1 holds consistently: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
- [methods] Benchmark reproduction with 100 traces: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] Pattern C2 > C1 holds across all 5 models tested: [evidence span withheld — anchor status: external_doc]
