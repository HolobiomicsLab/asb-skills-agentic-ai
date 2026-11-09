---
name: per-model-domain-benchmark-aggregation
description: Use when you have per-trace AgentLeak benchmark results for multiple
  LLM models and domains and need to verify or reproduce model-domain-level claims
  like the C2 >= C1 inequality.
when_to_use_negative:
- Input is not per-trace benchmark output (e.g., aggregated summary tables from the
  paper) — recompute rates from traces or compare directly instead
- Analyzing non-channel-level questions such as attack-family success rates (F2, F4,
  F6) or detection-mode latency (HYBRID ~200ms), which need different aggregations
tools:
- name: agentleak
  role: Benchmark package that generates the per-trace leakage results loaded and
    aggregated by this skill (installed via 'pip install -e .')
  repo: github.com/Privatris/AgentLeak
- name: GPT-4o
  role: One of the five production LLMs whose traces populate the model axis of the
    aggregation grid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:444598ea2563cb85e675d13e73f4200e9447c093d7ea6a2e2c6ac766eef19b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-4o
  - agentleak
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# per-model-domain-benchmark-aggregation

## Summary

Aggregates channel-level leakage counts from AgentLeak benchmark traces into per-model, per-domain leakage-rate tables (e.g., C1 final output vs C2 inter-agent messages) and evaluates cell-level claims such as C2 >= C1 across the 5 models x 4 domains grid.

## When to use

Use when you have per-trace AgentLeak benchmark results for multiple LLM models and domains and need to verify or reproduce model-domain-level claims like the C2 >= C1 inequality.

## When NOT to use

- Input is not per-trace benchmark output (e.g., aggregated summary tables from the paper) — recompute rates from traces or compare directly instead
- Analyzing non-channel-level questions such as attack-family success rates (F2, F4, F6) or detection-mode latency (HYBRID ~200ms), which need different aggregations

## Inputs

- AgentLeak trace files from benchmarks/ieee_repro/results/traces/ (produced by benchmark.py with --traces)
- Model identifiers for the five evaluated LLMs (e.g., openai/gpt-4o-mini via OpenRouter)
- Channel labels C1 (final output) and C2 (inter-agent messages) from the AgentLeak 7-channel taxonomy

## Outputs

- c2_geq_c1_verification.csv with columns (model, domain, c1_rate, c2_rate, c2_geq_c1) covering 20 model-domain cells
- Overall proportion of model-domain cells satisfying the C2 >= C1 inequality (expected 100%)

## How to apply

Load the reproduced benchmark results from the traces directory benchmarks/ieee_repro/results/traces/ produced by the agentleak package (e.g. via 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini'). Aggregate the channel-level leakage counts with pandas to compute per-model, per-domain leakage rates for channel C1 (final output) and channel C2 (inter-agent messages) across the 5 models (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) and 4 domains (healthcare, finance, legal, corporate). For each of the 20 model-domain cells, evaluate the inequality C2_rate >= C1_rate and record the cell-level pass/fail outcome. Write the resulting table (model, domain, c1_rate, c2_rate, c2_geq_c1) to c2_geq_c1_verification.csv and report the overall proportion of cells satisfying the inequality. The paper's acceptance criterion is a 100% pass rate over the 20 cells to reproduce the claim that 'C2 >= C1 holds consistently across all five models and four domains'.

## Related tools

- **agentleak** (Benchmark package that generates the per-trace leakage results loaded and aggregated by this skill (installed via 'pip install -e .')) — github.com/Privatris/AgentLeak
- **GPT-4o** (One of the five production LLMs whose traces populate the model axis of the aggregation grid)

## Examples

```
python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- The output table contains exactly 20 rows (5 models x 4 domains) with non-null c1_rate and c2_rate values
- Every row's c2_geq_c1 flag equals (c2_rate >= c1_rate) with no failures, matching the paper's 100% pass acceptance criterion
- Aggregate rates should be in the same regime as published numbers (C1 average ~27.2% in the paper; ~28.2% in the README) — large deviations indicate aggregation or trace-loading errors
- Traces must originate from the documented path benchmarks/ieee_repro/results/traces/ generated with benchmark.py --traces
- The overall pass proportion reported matches the per-row flags in the CSV (internal consistency check)

## Limitations

- The C2 >= C1 pattern is an empirical regularity over the evaluated models and domains, not a guarantee for unseen models or domains
- Cell-level rates depend on trace count per run (e.g., --n 100 vs --n 1000); small n makes rates noisy near the C1/C2 boundary
- The inequality check covers only channels C1 and C2; other channels (C3-C7, e.g. tool inputs C3 and system logs C6) are outside this verification and require benchmark_tools.py

## Evidence

- [abstract] Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
- [abstract] Inter-agent messages (C2) leak at 68.8%, versus 27.2% for final outputs (C1): "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [readme] Traces are in benchmarks/ieee_repro/results/traces/: [evidence span withheld — anchor status: external_doc]
- [methods] python benchmark.py --n 100 --traces --model openai/gpt-4o-mini: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
