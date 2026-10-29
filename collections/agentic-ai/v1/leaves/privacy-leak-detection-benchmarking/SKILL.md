---
name: privacy-leak-detection-benchmarking
description: Use when evaluating privacy leakage in multi-agent LLM pipelines — e.g.,
  auditing whether inter-agent messages (C2) leak more than final outputs (C1) across
  models like GPT-4o or Claude 3.5 Sonnet on scenarios in healthcare, finance, legal,
  or corporate domains.
when_to_use_negative:
- You need to redact or sanitize outputs rather than measure leakage — use the OutputSanitizer
  or PresidioDefense filter skills instead
- You are testing only a single agent with no inter-agent channels, where the audit-gap
  finding (C2 68.8% vs C1 27.2%) does not apply
- You cannot access the five production LLMs via OpenRouter and only need the static
  benchmark dataset without detection
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3489
tools:
- name: agentleak
  role: Core framework SDK and CLI for running benchmark scenarios, validation passes,
    and full benchmark runs with defenses
  repo: https://github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Detection engine instantiated with DetectionMode.HYBRID; check() and batch_check()
    evaluate vault/output/channel triples per scenario
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o
  role: One of the five production LLMs benchmarked in single-agent and multi-agent
    configurations
- name: GPT-4o-mini
  role: Default model for traced benchmark reproduction runs (openai/gpt-4o-mini)
- name: Claude 3.5 Sonnet
  role: Benchmarked model showing the paradox of lowest C1 leakage (8.2%) with the
    highest internal/external ratio (6.6x)
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_007
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:323c641981484ee06e06d194c07fdf11141fcc0a023fc418cedc4663d4344f4f
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
  - OutputSanitizer
  - agentleak
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# privacy-leak-detection-benchmarking

## Summary

Benchmark and detect privacy leakage across multi-agent LLM system channels using the AgentLeak framework, measuring leakage rates on final outputs (C1), inter-agent messages (C2), and memory (C5) against paper-reported baselines.

## When to use

Use when evaluating privacy leakage in multi-agent LLM pipelines — e.g., auditing whether inter-agent messages (C2) leak more than final outputs (C1) across models like GPT-4o or Claude 3.5 Sonnet on scenarios in healthcare, finance, legal, or corporate domains.

## When NOT to use

- You need to redact or sanitize outputs rather than measure leakage — use the OutputSanitizer or PresidioDefense filter skills instead
- You are testing only a single agent with no inter-agent channels, where the audit-gap finding (C2 68.8% vs C1 27.2%) does not apply
- You cannot access the five production LLMs via OpenRouter and only need the static benchmark dataset without detection

## Inputs

- Vault secret dictionaries (e.g., {"ssn": "123-45-6789"}) paired with candidate output text and a channel identifier (C1-C7)
- AgentLeak scenario set: 1,000 scenarios across healthcare, finance, legal, and corporate domains
- LLM outputs and inter-agent messages from single-agent and multi-agent coordinator-worker runs across the five production LLMs
- OpenRouter API key in .env for model access

## Outputs

- Per-check leak verdicts with confidence scores (result.leaked, result.confidence)
- Per-model leakage rates for C1 (single-agent vs multi-agent), C2, and C5 aggregated into leakage_rates.csv
- Per-mode latency and detection outcomes in detection_mode_results.csv
- Raw traces in benchmarks/ieee_repro/results/traces/ when run with --traces

## How to apply

Install the AgentLeak package locally ('pip install -e .' from the cloned Privatris/AgentLeak repo), place the OpenRouter API key in a .env file, and first run 'python -m agentleak run --quick --dry-run' to validate the harness. Then instantiate AgentLeakTester with DetectionMode.HYBRID — the ~200ms production-designated mode in the FAST (~10ms) / STANDARD (~100ms) / HYBRID (~200ms) / LLM_ONLY (~2s) latency profile — and call tester.check() or tester.batch_check() passing the vault secrets, the output text, and the channel identifier (C1 output, C2 inter-agent, C5 memory). For full reproduction, run 'python -m agentleak run --full' across the five production LLMs (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) in both single-agent and multi-agent coordinator-worker configurations, optionally with 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini' to capture per-scenario traces into benchmarks/ieee_repro/results/traces/. Aggregate per-model leakage rates for single-agent C1, multi-agent C1, and C2, and validate against the reported patterns: single-agent C1 43.2%, multi-agent C1 27.2%, C2 68.8%, with C2 >= C1 holding across all five models and four domains.

## Related tools

- **agentleak** (Core framework SDK and CLI for running benchmark scenarios, validation passes, and full benchmark runs with defenses) — https://github.com/Privatris/AgentLeak
- **AgentLeakTester** (Detection engine instantiated with DetectionMode.HYBRID; check() and batch_check() evaluate vault/output/channel triples per scenario) — https://github.com/Privatris/AgentLeak
- **GPT-4o** (One of the five production LLMs benchmarked in single-agent and multi-agent configurations)
- **GPT-4o-mini** (Default model for traced benchmark reproduction runs (openai/gpt-4o-mini))
- **Claude 3.5 Sonnet** (Benchmarked model showing the paradox of lowest C1 leakage (8.2%) with the highest internal/external ratio (6.6x))

## Examples

```
python -m agentleak run --full ; python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- Measured latency per DetectionMode matches the reported profile: FAST ~10ms, STANDARD ~100ms, HYBRID ~200ms, LLM_ONLY ~2s
- Reproduced C2 >= C1 pattern holds consistently across all five models and four domains
- Aggregated rates approximate reported values: single-agent C1 43.2%, multi-agent C1 27.2%, C2 68.8% (audit gap 41.7%)
- Validation pass 'python -m agentleak run --quick --dry-run' completes before any real scenario execution
- README per-model table reproduces qualitatively (e.g., Mistral-Large highest total leak at 99.3%, Claude-3.5-Sonnet lowest C1 at 8.2%)

## Limitations

- Output-only audits miss 41.7% of violations (README reports 45.9% audit gap), so C1-only measurement underestimates total exposure
- Total system exposure aggregated across C1, C2, C5 reaches 68.9% in the abstract but 79.7% in the README's 5,694-trace table — figures differ between sources
- Tool channels C3 and system logs C6 require separate runs via benchmark_tools.py (up to 85% leakage on Claude 3.5) and are not covered by the main benchmark.py path

## Evidence

- [abstract] Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [methods] HYBRID detection mode latency and production designation: "HYBRID | ~200ms | Production"
- [methods] Traced benchmark reproduction command: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [readme] Usage example of the tester with vault, output, and channel: "tester = AgentLeakTester(mode=DetectionMode.HYBRID)
result = tester.check(
    vault={"ssn": "123-45-6789"},
    output="The SSN is 123-45-6789",
    channel="C1"
)"
- [readme] README key result on internal vs external leakage ratio: [evidence span withheld — anchor status: external_doc]
