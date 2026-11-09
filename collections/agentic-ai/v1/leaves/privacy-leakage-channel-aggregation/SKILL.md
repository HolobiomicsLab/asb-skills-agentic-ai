---
name: privacy-leakage-channel-aggregation
description: Use when you have multi-agent LLM benchmark traces scored per channel
  and need total system privacy exposure as the union of C1/C2/C5 violations, plus
  the fraction of violations a final-output-only audit would miss.
when_to_use_negative:
- You are auditing only sanitized final outputs (C1) — this skill exists precisely
  because C1-only audits miss 41.7% of violations on internal channels
- You are scoring tool-input (C3) or system-log (C6) secondary-channel leakage — use
  the benchmark_tools.py advanced benchmark instead
- Your traces come from single-agent configurations — the C2 inter-agent channel requires
  multi-agent orchestration traces (CrewAI/LangGraph)
tools:
- name: agentleak
  role: Core benchmark SDK installed via `pip install -e .`; provides the traces and
    detection pipeline used for per-channel scoring
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Scores each trace's channel outputs against the vault with mode=DetectionMode.HYBRID
    (~200ms latency) to flag leakage on C1, C2, and C5
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o-mini
  role: Model under test for generating the reproduction traces (`--model openai/gpt-4o-mini`)
- name: CrewAI
  role: Multi-agent orchestration framework whose inter-agent messages constitute
    the C2 channel traces
- name: LangGraph
  role: Alternative multi-agent orchestration framework producing internal-channel
    (C2/C5) traces
- name: pandas
  role: Aggregates per-channel leakage rates across models and domains and computes
    the union of C1/C2/C5 violations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:2cef7a97d2e9cf7b263d422140f58ef04c9b2669cb2ed77a89e3fe43ace651d0
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
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# privacy-leakage-channel-aggregation

## Summary

Aggregates privacy-leakage detections across the final-output (C1), inter-agent message (C2), and memory-write (C5) channels of multi-agent LLM benchmark traces to compute total system exposure and the output-only audit miss fraction (41.7%). This quantifies violations invisible to C1-only audits because internal channels leak far more than final outputs.

## When to use

Use when you have multi-agent LLM benchmark traces scored per channel and need total system privacy exposure as the union of C1/C2/C5 violations, plus the fraction of violations a final-output-only audit would miss.

## When NOT to use

- You are auditing only sanitized final outputs (C1) — this skill exists precisely because C1-only audits miss 41.7% of violations on internal channels
- You are scoring tool-input (C3) or system-log (C6) secondary-channel leakage — use the benchmark_tools.py advanced benchmark instead
- Your traces come from single-agent configurations — the C2 inter-agent channel requires multi-agent orchestration traces (CrewAI/LangGraph)

## Inputs

- Multi-agent benchmark traces from benchmarks/ieee_repro/results/traces/ (generated with benchmark.py --n 100 --traces --model openai/gpt-4o-mini)
- Per-trace scenario vaults (e.g. {'ssn': '123-45-6789'}) used as ground truth for leakage detection
- Channel labels: C1 (External, final output), C2 (Internal, inter-agent messages), C5 (Internal, memory writes)

## Outputs

- Per-channel leakage rates across models and domains (C1, C2, C5)
- Total system exposure as the union of C1/C2/C5 violations (target: 68.9%)
- Output-only audit miss fraction (target: 41.7%, i.e. 68.8% C2 minus 27.2% C1)
- channel_exposure_aggregation.csv with per-channel rates, total exposure, and audit-miss fraction

## How to apply

Run the multi-agent benchmark (e.g. `python benchmark.py --n 100 --traces --model openai/gpt-4o-mini`) and load the traces from `benchmarks/ieee_repro/results/traces/`. Score each trace's channel outputs against the scenario vault using AgentLeakTester with DetectionMode.HYBRID (~200ms latency, designated for production), flagging leakage on channel C1 (External, final output), C2 (Internal, inter-agent messages), and C5 (Internal, memory writes). Aggregate per-channel leakage rates across models and domains with pandas, computing total system exposure as the union of violations across C1, C2, and C5. Recompute the output-only audit miss fraction as the excess of inter-agent (C2) leakage over final-output (C1) leakage: 68.8% − 27.2% = 41.7 percentage points. Write per-channel rates, total exposure, and audit-miss fraction to channel_exposure_aggregation.csv; the recomputed values must match the reported 68.9% total exposure and 41.7% audit miss.

## Related tools

- **agentleak** (Core benchmark SDK installed via `pip install -e .`; provides the traces and detection pipeline used for per-channel scoring) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Scores each trace's channel outputs against the vault with mode=DetectionMode.HYBRID (~200ms latency) to flag leakage on C1, C2, and C5) — https://github.com/Privatris/AgentLeak
- **GPT-4o-mini** (Model under test for generating the reproduction traces (`--model openai/gpt-4o-mini`))
- **CrewAI** (Multi-agent orchestration framework whose inter-agent messages constitute the C2 channel traces)
- **LangGraph** (Alternative multi-agent orchestration framework producing internal-channel (C2/C5) traces)
- **pandas** (Aggregates per-channel leakage rates across models and domains and computes the union of C1/C2/C5 violations)

## Examples

```
cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- Recomputed total system exposure across C1, C2, C5 must match the reported 68.9%
- Recomputed audit-miss fraction must equal the reported 41.7 percentage points (68.8% C2 minus 27.2% C1)
- Per-channel rates must reproduce C2 leakage at 68.8% and C1 final-output leakage at 27.2%
- The C2 >= C1 pattern must hold consistently across all five models and four domains in the aggregated results
- channel_exposure_aggregation.csv must contain per-channel rates, total exposure, and audit-miss fraction with consistent union-based aggregation (no double-counting of traces leaked on multiple channels)

## Limitations

- Reported headline rates (68.9% total, 68.8% C2, 27.2% C1) come from the IEEE repro run with 100 traces per model; the full 1,000-scenario README table reports different per-model values (e.g. average C1 28.2%, C2 74.0%), so expected numbers depend on run size
- Internal-channel leakage beyond C1/C2/C5 — tool inputs (C3) and system logs (C6) — can reach up to 85% (Claude 3.5) and is NOT captured by this C1/C2/C5 aggregation
- Detection latency of ~200ms per check in HYBRID mode means large trace sets require batch_check for throughput

## Evidence

- [abstract] Inter-agent messages leak far more than final outputs: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [abstract] Total system exposure raised by internal channels: "but introduce internal channels that raise total system exposure to 68.9% (aggregated across C1, C2, C5)"
- [methods] Benchmark reproduction command for traces: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] Location of generated traces for aggregation: [evidence span withheld — anchor status: external_doc]
- [readme] Detection mode designated for scoring: "tester = AgentLeakTester(mode=DetectionMode.HYBRID)"
