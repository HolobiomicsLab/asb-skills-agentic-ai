---
name: privacy-leakage-metric-analysis
description: Use when reproducing or auditing AgentLeak channel-level leakage results
  and you need to check whether the C2 >= C1 inequality holds per model-domain cell
  rather than only in aggregate.
when_to_use_negative:
- When analyzing other channels (C3 tool inputs, C5 memory, C6 logs) — those require
  the separate benchmark_tools.py pipeline, not the C1-vs-C2 inequality check
- When comparing single-agent vs multi-agent C1 leakage (27.2% vs 43.2%) — that is
  a different aggregate analysis, not the per-cell C2 >= C1 verification
- When only aggregate per-model rates are needed (e.g. README's C1/C2 summary table)
  without cell-level inequality evaluation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: agentleak
  role: Produces the channel-level benchmark traces consumed by this analysis; installed
    via 'pip install -e .'
  repo: github.com/Privatris/AgentLeak
- name: pytest
  role: Validates the agentleak package before running the benchmark whose traces
    feed the inequality analysis
  repo: github.com/Privatris/AgentLeak
- name: GPT-4o
  role: One of the five evaluated models whose traces are aggregated per-domain for
    the C1/C2 comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:9ac7f192967bc3275987204d7e10f5682d128557df2d9d96566484bc6edbe859
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GPT-4o
  - agentleak
  - pytest
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# privacy-leakage-metric-analysis

## Summary

Aggregates channel-level privacy leakage rates from AgentLeak benchmark traces and verifies the paper's claim that inter-agent message leakage (C2) meets or exceeds final-output leakage (C1) in every model-domain cell (5 models x 4 domains = 20 cells).

## When to use

Use when reproducing or auditing AgentLeak channel-level leakage results and you need to check whether the C2 >= C1 inequality holds per model-domain cell rather than only in aggregate.

## When NOT to use

- When analyzing other channels (C3 tool inputs, C5 memory, C6 logs) — those require the separate benchmark_tools.py pipeline, not the C1-vs-C2 inequality check
- When comparing single-agent vs multi-agent C1 leakage (27.2% vs 43.2%) — that is a different aggregate analysis, not the per-cell C2 >= C1 verification
- When only aggregate per-model rates are needed (e.g. README's C1/C2 summary table) without cell-level inequality evaluation

## Inputs

- Channel-level benchmark traces from benchmarks/ieee_repro/results/traces/ (per-trace leakage flags for C1 output and C2 inter-agent channels, per model and per domain)
- Model and domain identifiers for the 5 models (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) and 4 domains (healthcare, finance, legal, corporate)

## Outputs

- c2_geq_c1_verification.csv with columns (model, domain, c1_rate, c2_rate, c2_geq_c1) for all 20 model-domain cells
- Overall proportion of cells satisfying the C2 >= C1 inequality (expected: 100%)

## How to apply

Load the reproduced benchmark traces from benchmarks/ieee_repro/results/traces/ (produced via 'python benchmark.py --n 100 --traces --model <model>'). Use pandas to aggregate channel-level leakage counts into per-model, per-domain leakage rates for channel C1 (final output) and channel C2 (inter-agent messages) across the 5 models (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) and 4 domains (healthcare, finance, legal, corporate). For each of the 20 model-domain cells, evaluate the inequality c2_rate >= c1_rate and record a pass/fail outcome. Write the table (model, domain, c1_rate, c2_rate, c2_geq_c1) to c2_geq_c1_verification.csv and report the proportion of cells satisfying the inequality; the paper's acceptance criterion is a 100% pass rate over all 20 cells.

## Related tools

- **agentleak** (Produces the channel-level benchmark traces consumed by this analysis; installed via 'pip install -e .') — github.com/Privatris/AgentLeak
- **pytest** (Validates the agentleak package before running the benchmark whose traces feed the inequality analysis) — github.com/Privatris/AgentLeak
- **GPT-4o** (One of the five evaluated models whose traces are aggregated per-domain for the C1/C2 comparison)

## Examples

```
cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini && python -c "import pandas as pd; df=pd.read_csv('results/traces/*.csv'); df.groupby(['model','domain']).apply(lambda g: g[g.channel=='C2'].leaked.mean()>=g[g.channel=='C1'].leaked.mean())"
```

## Evaluation signals

- c2_geq_c1_verification.csv contains exactly 20 rows (5 models x 4 domains) with non-missing c1_rate and c2_rate
- The c2_geq_c1 column is True for all 20 cells, matching the paper's claim that 'Across all five models and four domains, the pattern C2 >= C1 holds consistently'
- Per-model C2 rates are well above C1 rates (e.g. README shows C2 internal leakage 2.6x C1 output leakage: 74.0% vs 28.2% average), so near-equality in any cell warrants inspection
- Recomputed rates are stable when the benchmark is re-run with the same --n trace count (e.g. 100 traces per model)

## Limitations

- The verification depends on trace volume; the README reproduction command uses --n 100 (or --n 1000 for full reproduction), so small samples may produce noisy per-cell rates
- The claim covers only channels C1 and C2; total system exposure (68.9% aggregated across C1, C2, C5) and secondary channels C3/C6 (up to 85% leakage) are outside this check
- Mistral-Large shows very high absolute leakage (C2 96.2%), so per-cell pass/fail can saturate and mask rate differences between models

## Evidence

- [abstract] C2 >= C1 pattern across models and domains: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
- [abstract] channel rates motivation: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [methods] reproduction command: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] traces location: [evidence span withheld — anchor status: external_doc]
- [readme] per-model rates: [evidence span withheld — anchor status: external_doc]
