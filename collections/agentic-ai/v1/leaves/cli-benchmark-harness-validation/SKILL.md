---
name: cli-benchmark-harness-validation
description: Use when reproducing AgentLeak leakage-rate results or preparing a fresh
  benchmark execution where the harness, OpenRouter API key, and model configurations
  must be verified before committing to costly full runs.
when_to_use_negative:
- The task is measuring leakage on a single model's response string rather than running
  the multi-model benchmark harness — use the AgentLeakTester SDK directly (tester.check
  / batch_check) instead
- The target system is not an LLM agent pipeline with internal channels C1-C7; the
  harness assumes agent scenarios with inter-agent messages, memory, and tool channels
- You need defense-specific results without first validating the harness — skip is
  unsafe; the workflow requires the dry-run validation pass before real scenario execution
edam_operation: http://edamontology.org/operation_2428
tools:
- name: agentleak
  role: CLI harness and SDK installed via 'pip install -e .' that runs --quick --dry-run
    validation and --full benchmark execution
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Detection component instantiated with DetectionMode.HYBRID to check leakage
    on channels C1 and C2 during benchmark runs
  repo: https://github.com/Privatris/AgentLeak
- name: pytest
  role: Runs the repository test suite ('pytest tests/ -v') as a pre-run harness check
  repo: https://github.com/Privatris/AgentLeak
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:aabaf494262c07762e50e1e236f5476fe9e4508919db20f2ae71b9d937b412d7
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
  - pytest
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cli-benchmark-harness-validation

## Summary

Validates and runs the AgentLeak CLI benchmark harness: a quick dry-run pass confirms the installed package and API-key configuration before executing the full multi-channel privacy-leakage benchmark (C1, C2, C5) across five production LLMs in single- and multi-agent modes.

## When to use

Use when reproducing AgentLeak leakage-rate results or preparing a fresh benchmark execution where the harness, OpenRouter API key, and model configurations must be verified before committing to costly full runs.

## When NOT to use

- The task is measuring leakage on a single model's response string rather than running the multi-model benchmark harness — use the AgentLeakTester SDK directly (tester.check / batch_check) instead
- The target system is not an LLM agent pipeline with internal channels C1-C7; the harness assumes agent scenarios with inter-agent messages, memory, and tool channels
- You need defense-specific results without first validating the harness — skip is unsafe; the workflow requires the dry-run validation pass before real scenario execution

## Inputs

- AgentLeak repository checkout with agentleak installed via 'pip install -e .'
- OpenRouter API key in .env
- agentleak_data/ scenario dataset (1,000 scenarios across healthcare, finance, legal, corporate domains)
- CLI flags selecting run mode (--quick --dry-run / --full), attack family (--attack-family F4), defense (--defense D1), and model (e.g. openai/gpt-4o-mini)

## Outputs

- Per-model leakage rates for C1 single-agent, C1 multi-agent, and C2 aggregated into leakage_rates.csv
- Trace files written to benchmarks/ieee_repro/results/traces/ (and results/tools/ for the C3/C6 tools-and-logs benchmark)
- Console pass/fail confirmation that the harness is correctly set up from the dry-run

## How to apply

Clone github.com/Privatris/AgentLeak, install the agentleak package in editable mode with 'pip install -e .', and write the OpenRouter key into .env. First run the test suite ('pytest tests/ -v') and the validation pass 'python -m agentleak run --quick --dry-run' to confirm the harness is correctly set up without executing real scenarios. Once validated, execute the full benchmark with 'python -m agentleak run --full' (or a traced run 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini') across GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, and Llama 3.3 70B in both single-agent and multi-agent coordinator-worker configurations, using AgentLeakTester with DetectionMode.HYBRID on channels C1 and C2. Aggregate per-model C1 (single-agent), C1 (multi-agent), and C2 rates into leakage_rates.csv. The rationale is that a dry-run catches setup errors cheaply, since full runs span thousands of traces (5,694 traces across 5 models in the paper) and traced output is written to benchmarks/ieee_repro/results/traces/.

## Related tools

- **agentleak** (CLI harness and SDK installed via 'pip install -e .' that runs --quick --dry-run validation and --full benchmark execution) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Detection component instantiated with DetectionMode.HYBRID to check leakage on channels C1 and C2 during benchmark runs) — https://github.com/Privatris/AgentLeak
- **pytest** (Runs the repository test suite ('pytest tests/ -v') as a pre-run harness check) — https://github.com/Privatris/AgentLeak

## Examples

```
python -m agentleak run --quick --dry-run
```

## Evaluation signals

- The 'python -m agentleak run --quick --dry-run' pass completes without real scenario execution, confirming the harness and .env API-key setup
- 'pytest tests/ -v' passes before any benchmark run
- Aggregated leakage_rates.csv reproduces the ordering C2 >= C1 consistently across all five models
- Reproduced rates approximate the reported values: single-agent C1 43.2%, multi-agent C1 27.2%, C2 68.8%, audit gap 41.7%
- Traced runs populate benchmarks/ieee_repro/results/traces/ with the requested number of traces (--n 100)

## Limitations

- Per-model rates in the README (average C1 28.2%, C2 74.0%) differ from the abstract's headline figures (C1 27.2%/43.2%, C2 68.8%), so expected values depend on whether single-agent vs multi-agent splits are aggregated
- Full runs require an OpenRouter API key and incur real model calls; the dry-run is dry by design and cannot validate model-dependent behavior
- The repository shows no changelog ('No changelog found.'), so harness behavior may shift without version notice
- Detection latency for HYBRID mode is ~200ms per check, designating it for production use but not for high-throughput sweeps

## Evidence

- [methods] Run the validation pass 'python -m agentleak run --quick --dry-run' to confirm the harness is correctly set up before executing real scenarios.: [evidence span withheld — anchor status: not_found]
- [methods] Install agentleak package and configure API key: "pip install -e .
echo "OPENROUTER_API_KEY=your_key" > .env"
- [readme] Traced reproduction run writes traces to benchmarks/ieee_repro/results/traces/: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
``n
Traces are in `benchmarks/ieee_repro/results/traces/`."
- [abstract] Validation ordering across all models: "Across all five models and four domains, the pattern C2 $\geq$ C1 holds consistently"
