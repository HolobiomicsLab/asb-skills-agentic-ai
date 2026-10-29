---
name: multi-agent-coordinator-worker-configuration
description: Use when evaluating whether multi-agent LLM coordination (e.g., CrewAI
  or LangGraph coordinator-worker pipelines) changes privacy leakage compared to a
  single-agent baseline, or when auditing internal channels beyond final outputs.
when_to_use_negative:
- When the question is only about final-output (C1) sanitization quality — the skill's
  core value is the C1 vs C2 audit gap, so an output-only audit will miss the inter-agent
  violations this skill targets
- When the system under test is not a coordinator-worker multi-agent pipeline (or
  single-agent baseline) instrumented with the agentleak SDK on CrewAI/LangGraph integrations
- When offline/deterministic evaluation is required — runs depend on live LLM API
  calls via OpenRouter and HYBRID detection (~200ms latency per check)
tools:
- name: agentleak
  role: Core benchmark harness and CLI used to run single-agent and multi-agent configurations
    across channels C1/C2
  repo: github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Leakage detector instantiated in DetectionMode.HYBRID to check outputs and
    inter-agent messages against a secret vault per channel
  repo: github.com/Privatris/AgentLeak
- name: CrewAI
  role: Multi-agent framework used to build the coordinator-worker configuration whose
    internal channels are audited
- name: LangGraph
  role: Alternative state-graph multi-agent framework instrumented via add_agentleak_to_langgraph
    for channel auditing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:4cc686b71b9b2bdbd88f1e4f15a419c2771f863c8c1752d1c259ef1c9c8405cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AgentLeakTester
  - Claude 3.5 Sonnet
  - CrewAI
  - GPT-4o
  - GPT-4o-mini
  - LangGraph
  - Llama 3.3 70B
  - Mistral Large
  - agentleak
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# multi-agent-coordinator-worker-configuration

## Summary

Sets up and benchmarks single-agent vs multi-agent coordinator-worker LLM configurations in the AgentLeak harness to quantify privacy leakage on final outputs (C1) versus internal inter-agent channels (C2). It exists because output-only audits miss a large fraction of violations that occur on internal channels.

## When to use

Use when evaluating whether multi-agent LLM coordination (e.g., CrewAI or LangGraph coordinator-worker pipelines) changes privacy leakage compared to a single-agent baseline, or when auditing internal channels beyond final outputs.

## When NOT to use

- When the question is only about final-output (C1) sanitization quality — the skill's core value is the C1 vs C2 audit gap, so an output-only audit will miss the inter-agent violations this skill targets
- When the system under test is not a coordinator-worker multi-agent pipeline (or single-agent baseline) instrumented with the agentleak SDK on CrewAI/LangGraph integrations
- When offline/deterministic evaluation is required — runs depend on live LLM API calls via OpenRouter and HYBRID detection (~200ms latency per check)

## Inputs

- AgentLeak scenario dataset (agentleak_data, 1,000 scenarios across healthcare, finance, legal, corporate domains)
- Secret vault definitions (e.g., {"ssn": "123-45-6789"}) supplied to AgentLeakTester.check
- OpenRouter API key for the five production LLMs (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B)

## Outputs

- Per-model leakage rates for C1 (single-agent), C1 (multi-agent), and C2 aggregated into leakage_rates.csv
- Per-check AgentLeakTester results (result.leaked, result.confidence) with channel tags C1/C2
- Optional run traces written to benchmarks/ieee_repro/results/traces/

## How to apply

Clone the AgentLeak repository (github.com/Privatris/AgentLeak), install with 'pip install -e .', set the OpenRouter API key in .env, and verify the harness with 'python -m agentleak run --quick --dry-run' before real runs. Execute 'python -m agentleak run --full' across the five production LLMs (GPT-4o, GPT-4o-mini, Claude 3.5 Sonnet, Mistral Large, Llama 3.3 70B) in both single-agent and multi-agent coordinator-worker configurations, detecting leakage with AgentLeakTester in DetectionMode.HYBRID on channels C1 and C2; optionally run 'python benchmark.py --n 100 --traces --model openai/gpt-4o-mini' in benchmarks/ieee_repro to capture traces. Aggregate per-model rates for C1 single-agent, C1 multi-agent, and C2 into leakage_rates.csv. Validate that C2 >= C1 holds across all five models and that rates approximate the reported values (single-agent C1 43.2%, multi-agent C1 27.2%, C2 68.8%).

## Related tools

- **agentleak** (Core benchmark harness and CLI used to run single-agent and multi-agent configurations across channels C1/C2) — github.com/Privatris/AgentLeak
- **AgentLeakTester** (Leakage detector instantiated in DetectionMode.HYBRID to check outputs and inter-agent messages against a secret vault per channel) — github.com/Privatris/AgentLeak
- **CrewAI** (Multi-agent framework used to build the coordinator-worker configuration whose internal channels are audited)
- **LangGraph** (Alternative state-graph multi-agent framework instrumented via add_agentleak_to_langgraph for channel auditing)

## Examples

```
python -m agentleak run --full  # single- vs multi-agent configs, channels C1/C2; or: python benchmark.py --n 100 --traces --model openai/gpt-4o-mini
```

## Evaluation signals

- Reproduced pattern C2 >= C1 holds consistently across all five models (and four domains) in the aggregated leakage_rates.csv
- Single-agent C1 leakage approximates 43.2%, multi-agent C1 approximates 27.2%, and C2 approximates 68.8% per the paper's reported table
- Validation pass 'python -m agentleak run --quick --dry-run' completes cleanly before real scenario execution
- Each AgentLeakTester.check result carries a leaked flag and confidence value tied to the correct channel string (C1 or C2)
- Audit gap computed as C2 minus C1 multi-agent (41.7 percentage points reported) matches the fraction of violations an output-only audit would miss

## Limitations

- Reported headline averages differ between sources: the paper card cites C1 43.2% (single), 27.2% (multi), C2 68.8%, while the README's 5,694-trace table lists averages of 28.2% (C1), 74.0% (C2), and 45.9% audit gap — expect model-level variance (e.g., Claude 3.5 Sonnet C1 8.2% vs C2 53.9%)
- Multi-agent configurations reduce final-output leakage but simultaneously raise total system exposure to 68.9% (aggregated across C1, C2, C5) via internal channels invisible to output-level defenses
- Results depend on live OpenRouter-hosted model behavior; the repository has no changelog ('No changelog found'), so version drift between paper and repo is possible

## Evidence

- [abstract] Multi-agent configs reduce C1 leakage vs single-agent baseline: "multi-agent configurations reduce final-output leakage (C1: 27.2% vs 43.2% in single-agent mode) compared with single-agent baselines"
- [abstract] C2 leaks at 68.8% vs 27.2% for C1; output-only audits miss 41.7%: "Inter-agent messages (C2) leak at 68.8%, compared with 27.2% for final outputs (C1), meaning that output-only audits miss 41.7% of violations"
- [readme] Internal channels leak 2.6x more than external across all 5 models: [evidence span withheld — anchor status: external_doc]
- [methods] HYBRID detection mode is the production configuration used for channel checks: "HYBRID | ~200ms | Production"
