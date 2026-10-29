---
name: multi-agent-coordination-attack-simulation
description: Use when benchmarking or stress-testing a multi-agent LLM pipeline's
  exposure to coordinated inter-agent privacy leakage, or when reproducing the AgentLeak
  family-level success-rate table (F1-F6).
when_to_use_negative:
- When the target system is a single-agent pipeline with no inter-agent communication
  channels — internal-channel attacks like F4 require C2 (inter-agent message) pathways
  to exploit
- When the goal is to measure only final-output (C1) leakage under output-only audit
  — F4's exposure signal lives in internal channels invisible to output-level defenses
- When evaluating defensive components (e.g., OutputSanitizer, PresidioDefense) rather
  than measuring attack success rates — use `python -m agentleak run --defense D1`
  instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: agentleak
  role: Core benchmark framework and CLI used to install, validate, and execute the
    F4 attack-family run (`--attack-family F4`)
  repo: https://github.com/Privatris/AgentLeak
- name: GPT-4o-mini
  role: Target model for trace-level reproduction via benchmark.py (`--model openai/gpt-4o-mini`)
- name: GPT-4o
  role: One of the five production LLMs evaluated in the AgentLeak benchmark; usable
    as target model for the family-level run
- name: CrewAI
  role: Multi-agent framework integrated via add_agentleak_to_crew to embed privacy
    checks in coordinated agent crews
- name: LangGraph
  role: Multi-agent framework integrated via add_agentleak_to_langgraph for state-graph-based
    agent pipelines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:a89ab1e5e80365283dd5c8b5f280ec9da477ec1fec0537994efd5c58c2d2c0f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Privatris/AgentLeak
  tools:
  - CrewAI
  - GPT-4o
  - GPT-4o-mini
  - LangGraph
  - agentleak
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-agent-coordination-attack-simulation

## Summary

Simulates multi-agent coordination attacks (AgentLeak attack family F4) against benchmark scenarios to measure how attackers exploit inter-agent communication channels in multi-agent LLM systems. F4 is the highest-success attack family in the AgentLeak taxonomy at 80%, making it the reference case for internal-channel privacy risk.

## When to use

Use when benchmarking or stress-testing a multi-agent LLM pipeline's exposure to coordinated inter-agent privacy leakage, or when reproducing the AgentLeak family-level success-rate table (F1-F6).

## When NOT to use

- When the target system is a single-agent pipeline with no inter-agent communication channels — internal-channel attacks like F4 require C2 (inter-agent message) pathways to exploit
- When the goal is to measure only final-output (C1) leakage under output-only audit — F4's exposure signal lives in internal channels invisible to output-level defenses
- When evaluating defensive components (e.g., OutputSanitizer, PresidioDefense) rather than measuring attack success rates — use `python -m agentleak run --defense D1` instead

## Inputs

- AgentLeak benchmark scenarios (1,000 scenarios across healthcare, finance, legal, corporate domains)
- OpenRouter API key written to .env
- Target model identifier (e.g., openai/gpt-4o-mini, anthropic/claude-3.5-sonnet)
- Vault of sensitive values (e.g., {"ssn": "123-45-6789"}) used for leak detection

## Outputs

- F4 (multi-agent coordination) attack success rate, expected at 80% per the reported family table
- Trace files written to benchmarks/ieee_repro/results/traces/
- Aggregated attack_family_success_rates.csv matching F1=62%, F2=72%, F3=63%, F4=80%, F5=63%, F6=68%

## How to apply

Install the agentleak package locally (`pip install -e .`), write the OpenRouter API key to `.env`, and validate the setup with `python -m agentleak run --quick --dry-run`. Run the full benchmark with `python -m agentleak run --full`, then isolate the F4 family with `python -m agentleak run --attack-family F4` to execute multi-agent coordination attacks against the 1,000 AgentLeak scenarios. For trace-level reproduction, run `cd benchmarks/ieee_repro && python benchmark.py --n 100 --traces --model openai/gpt-4o-mini`, which writes traces to `benchmarks/ieee_repro/results/traces/`. Aggregate per-family success rates and check F4 lands at 80% (matching the reported table) within run-to-run variance. Rationale: F4 exploits inter-agent messages (channel C2), which the paper shows leak at 68.8% versus 27.2% for final outputs (C1), so this family is the strongest probe of internal-channel vulnerability that output-only audits miss.

## Related tools

- **agentleak** (Core benchmark framework and CLI used to install, validate, and execute the F4 attack-family run (`--attack-family F4`)) — https://github.com/Privatris/AgentLeak
- **GPT-4o-mini** (Target model for trace-level reproduction via benchmark.py (`--model openai/gpt-4o-mini`))
- **GPT-4o** (One of the five production LLMs evaluated in the AgentLeak benchmark; usable as target model for the family-level run)
- **CrewAI** (Multi-agent framework integrated via add_agentleak_to_crew to embed privacy checks in coordinated agent crews)
- **LangGraph** (Multi-agent framework integrated via add_agentleak_to_langgraph for state-graph-based agent pipelines)

## Examples

```
python -m agentleak run --attack-family F4
```

## Evaluation signals

- Reproduced F4 success rate falls at 80% within run-to-run variance, matching the reported family table (F1=62%, F2=72%, F3=63%, F4=80%, F5=63%, F6=68%)
- `python -m agentleak run --quick --dry-run` completes without error, validating installation and API-key configuration before the full run
- Trace files are present in benchmarks/ieee_repro/results/traces/ after the --traces run
- Leak detections carry both a leaked boolean and a confidence value from AgentLeakTester.check (e.g., result.leaked, result.confidence) rather than bare verdicts
- Leakage attributed to internal channels (C2) exceeds final-output leakage (C1) in the F4 results, consistent with the paper's invariant C2 >= C1

## Limitations

- The 80% F4 figure is a reported table value; run-to-run variance means a single reproduction may deviate, so verification is 'within variance', not exact-match
- Results are model-dependent: internal-channel leakage varies widely across the five tested models (e.g., Claude 3.5 Sonnet C1=8.2% vs Mistral-Large total leak 99.3%), so F4 rates should not be extrapolated across models
- Benchmark uses OpenRouter-hosted models; the run requires a valid OPENROUTER_API_KEY in .env and network access
- The preprint is dated 2026 and the repo license is marked NOASSERTION in metadata despite an MIT notice in the README — check licensing before redistribution

## Evidence

- [methods] Multi-agent coordination attacks (F4) have a reported success rate of 80%: "F4 | Multi-agent coordination | 80%"
- [methods] Run specific attack family benchmark: "python -m agentleak run --attack-family F4   # Specific attacks"
- [methods] Benchmark reproduction with 100 traces: "python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
- [abstract] Inter-agent messages leak far more than final outputs, motivating F4 as the strongest internal-channel probe: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [readme] Reproduction command and trace output location from the README: "cd benchmarks/ieee_repro
python benchmark.py --n 100 --traces --model openai/gpt-4o-mini"
