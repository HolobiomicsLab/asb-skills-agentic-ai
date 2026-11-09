---
name: llm-trace-collection-and-replay
description: Use when reproducing or auditing AgentLeak benchmark results requires
  per-scenario traces (e.g. verifying channel-level leakage findings or inspecting
  individual multi-agent coordination failures) rather than only aggregate rates.
when_to_use_negative:
- You only need the aggregate headline numbers (C1 vs C2 rates) and can use the untraced
  'python -m agentleak run --full' mode, saving cost and time
- You are evaluating a non-multi-agent, single-model pipeline where internal-channel
  leakage (C2 inter-agent messages) is not in scope
- The agentleak package or OpenRouter API key is not yet installed/configured — run
  the '--quick --dry-run' validation instead of a traced run
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: agentleak
  role: Core benchmark SDK whose CLI executes scenarios and whose reproduction scripts
    (benchmark.py, benchmark_tools.py) emit per-scenario traces
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Hybrid-mode (DetectionMode.HYBRID) leakage detector applied per channel (C1,
    C2, C5) during traced runs to label each trace as leaked/not leaked with a confidence
    score
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o-mini
  role: Default replay model for the traced reproduction command (--model openai/gpt-4o-mini)
- name: GPT-4o
  role: One of the five production LLMs that can be replayed in traced runs to compare
    per-model channel leakage
- name: Claude 3.5 Sonnet
  role: Replay model of interest for the C3/C6 tool-leakage analysis (up to 85% leakage
    on tool inputs/logs) via benchmark_tools.py
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:aa1767c91789aa51c44c53f7cb7572ddb9136bd38e9f6061a30a781501f9a800
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# llm-trace-collection-and-replay

## Summary

Collect per-scenario LLM execution traces from the AgentLeak benchmark runs and replay them via the traced reproduction script so that leakage across channels (C1 output, C2 inter-agent, C5 memory, C3 tools, C6 logs) can be inspected and audited at the trace level rather than only aggregated.

## When to use

Use when reproducing or auditing AgentLeak benchmark results requires per-scenario traces (e.g. verifying channel-level leakage findings or inspecting individual multi-agent coordination failures) rather than only aggregate rates.

## When NOT to use

- You only need the aggregate headline numbers (C1 vs C2 rates) and can use the untraced 'python -m agentleak run --full' mode, saving cost and time
- You are evaluating a non-multi-agent, single-model pipeline where internal-channel leakage (C2 inter-agent messages) is not in scope
- The agentleak package or OpenRouter API key is not yet installed/configured — run the '--quick --dry-run' validation instead of a traced run

## Inputs

- OpenRouter API key written to .env
- agentleak repository checkout (github.com/Privatris/AgentLeak) with agentleak_data/ dataset of 1,000 scenarios
- Model identifier in OpenRouter format (e.g. openai/gpt-4o-mini, anthropic/claude-3.5-sonnet)
- Trace sample size n (100 for spot-checks, 1000 for full reproduction)

## Outputs

- Trace files written to benchmarks/ieee_repro/results/traces/
- Tool/log-channel benchmark results saved in benchmarks/ieee_repro/results/tools/
- Per-channel leakage rates (C1, C2, C5 and optionally C3, C6) aggregatable into tables such as leakage_rates.csv

## How to apply

After installing the agentleak package ('pip install -e .') and configuring the OpenRouter API key in .env, validate the harness with 'python -m agentleak run --quick --dry-run' before spending API budget. Then run the traced reproduction script from benchmarks/ieee_repro: 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini', which writes traces to benchmarks/ieee_repro/results/traces/; use --n 1000 to match the full paper scale (5,694 traces across 5 models). Choose the model explicitly via the OpenRouter identifier (e.g. openai/gpt-4o, anthropic/claude-3.5-sonnet). For tool/log channel analysis (C3, C6), replay with benchmark_tools.py --n 100 --model <model> or the run_tools_benchmark.sh multi-model suite, saving results to benchmarks/ieee_repro/results/tools/. Judge success by checking that trace files appear in results/traces/, that the C2 >= C1 pattern holds across models in the traced data, and that aggregated per-channel rates approximate the reported values (e.g. C1 28.2%, C2 74.0% on average).

## Related tools

- **agentleak** (Core benchmark SDK whose CLI executes scenarios and whose reproduction scripts (benchmark.py, benchmark_tools.py) emit per-scenario traces) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Hybrid-mode (DetectionMode.HYBRID) leakage detector applied per channel (C1, C2, C5) during traced runs to label each trace as leaked/not leaked with a confidence score) — https://github.com/Privatris/AgentLeak
- **GPT-4o-mini** (Default replay model for the traced reproduction command (--model openai/gpt-4o-mini))
- **GPT-4o** (One of the five production LLMs that can be replayed in traced runs to compare per-model channel leakage)
- **Claude 3.5 Sonnet** (Replay model of interest for the C3/C6 tool-leakage analysis (up to 85% leakage on tool inputs/logs) via benchmark_tools.py)

## Examples

```
cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- Trace files are produced under benchmarks/ieee_repro/results/traces/ with one trace per executed scenario
- The C2 >= C1 pattern holds consistently across all five replayed models in the traced data
- Aggregated traced rates approximate reported averages: C1 ~28.2%, C2 ~74.0%, total leak ~79.7%, audit gap ~45.9%
- Model-specific spot checks match the README table, e.g. Claude-3.5-Sonnet C1=8.2% vs C2=53.9%, GPT-4o C1=17.2% vs C2=76.8%
- Pre-run validation 'python -m agentleak run --quick --dry-run' completes without error before any real traced run

## Limitations

- Run-to-run variance means traced rates only approximate reported values; exact reproduction of headline percentages is not guaranteed
- Replaying large n (1000 scenarios across 5 models, 5,694 traces total) incurs substantial LLM API cost and runtime
- The C3 (tool inputs) and C6 (logs) channels require the separate benchmark_tools.py script, not the main benchmark.py, so a single replay does not cover all 7 channels

## Evidence

- [methods] Traced reproduction command: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] Trace output location: [evidence span withheld — anchor status: external_doc]
- [readme] Scale of traced data: [evidence span withheld — anchor status: external_doc]
- [methods] Validation before real runs: "python -m agentleak run --quick --dry-run    # Validation"
- [abstract] Cross-model invariant for traced data: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
