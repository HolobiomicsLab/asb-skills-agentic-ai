---
name: internal-vs-output-channel-audit-analysis
description: Use when evaluating privacy leakage in multi-agent LLM pipelines where
  traces contain internal channels (inter-agent messages, memory writes) and you must
  determine how much more leaks internally than externally, i.e., the output-only
  audit miss rate.
when_to_use_negative:
- Your system is single-agent only — internal channels C2/C5 do not exist; compare
  against the single-agent C1 baseline (43.2%) instead.
- You are auditing tool-inputs (C3) or system logs (C6) leakage (Finding 7) — use
  the separate benchmark_tools.py pipeline, not the C1/C2/C5 aggregation.
- Latency budget rules out HYBRID detection (~200ms per check) and you only need pattern-based
  sanitization checks (OutputSanitizer).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: agentleak
  role: Core benchmark/SDK package that provides the trace dataset, channel taxonomy,
    and detection pipeline
  repo: github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Scores each channel's output against the vault (mode=DetectionMode.HYBRID)
    to flag leakage on C1, C2, C5
  repo: github.com/Privatris/AgentLeak
- name: CrewAI
  role: Multi-agent framework used to generate inter-agent message traffic (C2) in
    benchmark scenarios
- name: LangGraph
  role: Multi-agent framework used to generate inter-agent message traffic (C2) in
    benchmark scenarios
- name: GPT-4o-mini
  role: Model under test for trace generation (e.g., --model openai/gpt-4o-mini)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:613c37ddba60b0a54a3784a3555ba44aaeeb27abcae41c5c8949e4fa642ad826
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AgentLeakTester
  - CrewAI
  - GPT-4o
  - GPT-4o-mini
  - LangGraph
  - agentleak
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# internal-vs-output-channel-audit-analysis

## Summary

Quantifies the privacy audit gap in multi-agent LLM systems by comparing inter-agent message (C2) leakage against final-output (C1) leakage, computing total system exposure across C1/C2/C5 and the fraction of violations invisible to output-only audits (e.g., 68.8% vs 27.2% → 41.7% missed).

## When to use

Use when evaluating privacy leakage in multi-agent LLM pipelines where traces contain internal channels (inter-agent messages, memory writes) and you must determine how much more leaks internally than externally, i.e., the output-only audit miss rate.

## When NOT to use

- Your system is single-agent only — internal channels C2/C5 do not exist; compare against the single-agent C1 baseline (43.2%) instead.
- You are auditing tool-inputs (C3) or system logs (C6) leakage (Finding 7) — use the separate benchmark_tools.py pipeline, not the C1/C2/C5 aggregation.
- Latency budget rules out HYBRID detection (~200ms per check) and you only need pattern-based sanitization checks (OutputSanitizer).

## Inputs

- Multi-agent benchmark traces in benchmarks/ieee_repro/results/traces/ (JSON trace files from benchmark.py --traces)
- Privacy vault of sensitive values (e.g., {"ssn": "123-45-6789"}) per scenario
- Model configuration (e.g., openai/gpt-4o-mini via OPENROUTER_API_KEY)

## Outputs

- Per-channel leakage rates (C1, C2, C5) aggregated across models and domains
- Total system exposure (union of C1/C2/C5 violations, e.g., 68.9%)
- Output-only audit miss fraction / audit gap H1 (C2 - C1, e.g., 41.7%)
- channel_exposure_aggregation.csv

## How to apply

Run the multi-agent benchmark (e.g., 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini') to generate traces in benchmarks/ieee_repro/results/traces/. Score each trace's channel outputs against the vault with AgentLeakTester in DetectionMode.HYBRID (~200ms latency, designated for production use), flagging leakage on C1 (external final output), C2 (internal inter-agent messages), and C5 (internal memory writes). Aggregate per-channel rates with pandas, computing total system exposure as the union of violations across C1, C2, and C5. Recompute the audit gap H1 as C2 minus C1 (e.g., 68.8% - 27.2% = 41.7 percentage points), and verify the union exposure (e.g., 68.9%) matches the reported value. The rationale is that C2 >= C1 holds consistently across all five models and four domains, so output-only defenses systematically undercount risk.

## Related tools

- **agentleak** (Core benchmark/SDK package that provides the trace dataset, channel taxonomy, and detection pipeline) — github.com/Privatris/AgentLeak
- **AgentLeakTester** (Scores each channel's output against the vault (mode=DetectionMode.HYBRID) to flag leakage on C1, C2, C5) — github.com/Privatris/AgentLeak
- **CrewAI** (Multi-agent framework used to generate inter-agent message traffic (C2) in benchmark scenarios)
- **LangGraph** (Multi-agent framework used to generate inter-agent message traffic (C2) in benchmark scenarios)
- **GPT-4o-mini** (Model under test for trace generation (e.g., --model openai/gpt-4o-mini))

## Examples

```
python benchmark.py --n 100 --traces --model openai/gpt-4o-mini; tester = AgentLeakTester(mode=DetectionMode.HYBRID); result = tester.check(vault={'ssn': '123-45-6789'}, output='The SSN is 123-45-6789', channel='C1')
```

## Evaluation signals

- Recomputed total system exposure (union of C1, C2, C5 violations) must match the reported 68.9%.
- Recomputed audit-miss fraction must equal C2 - C1, i.e., 68.8% - 27.2% = 41.7 percentage points.
- The inequality C2 >= C1 must hold for every model and every domain tested; any violation flags a scoring or channel-attribution error.
- Per-model results should reproduce README ordering, e.g., GPT-4o: C1=17.2%, C2=76.8%, H1=59.6%; Claude-3.5-Sonnet: C1=8.2%, C2=53.9%, H1=45.7%.
- channel_exposure_aggregation.csv must contain per-channel rates, total exposure, and audit-miss fraction with consistent denominators (all scored traces).

## Limitations

- Output-only audits (C1-only) miss 41.7% of violations, so C1-based numbers alone are systematically biased downward.
- The Claude 3.5 Sonnet paradox shows C1 can be very low (8.2%) while internal/external ratio is highest (6.6×), so C1 is a poor proxy for internal risk.
- Aggregate figures depend on the 5,694-trace corpus across 5 models and 4 domains (healthcare, finance, legal, corporate); other models/domains require fresh runs.
- HYBRID detection adds ~200ms latency per check; DetectionMode choice affects results.

## Evidence

- [abstract] Inter-agent messages (C2) leak at 68.8%, versus 27.2% for final outputs (C1), so output-only audits miss 41.7% of violations: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [abstract] Total system exposure reaches 68.9% across C1/C2/C5: "but introduce internal channels that raise total system exposure to 68.9% (aggregated across C1, C2, C5)"
- [methods] HYBRID detection mode latency and production designation: "HYBRID | ~200ms | Production"
- [abstract] C2 >= C1 pattern consistency across models and domains: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
- [readme] Reproduction command generating traces: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
