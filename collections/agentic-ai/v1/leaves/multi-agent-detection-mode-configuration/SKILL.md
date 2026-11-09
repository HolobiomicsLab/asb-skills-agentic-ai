---
name: multi-agent-detection-mode-configuration
description: Use when deploying or benchmarking AgentLeakTester against multi-agent
  LLM traces and you must pick a DetectionMode whose latency budget (FAST ~10ms to
  LLM_ONLY ~2s) and role (HYBRID = production) match the evaluation channel.
when_to_use_negative:
- You only need a single-model external-output audit without latency profiling — the
  full mode sweep over FAST/STANDARD/HYBRID/LLM_ONLY is unnecessary overhead
- Your system has no multi-agent internal channels (C2 inter-agent, C5 memory, C3-C4
  tools) — the mode designations were characterized for internal-channel leakage in
  multi-agent systems
- Latency tolerance is far below ~10ms per check — even the FAST mode's profile makes
  it unsuitable for sub-10ms inline filtering
tools:
- name: agentleak
  role: Core framework SDK providing the tester, DetectionMode enum, and CLI (`python
    -m agentleak run`) used to sweep and evaluate detection modes
  repo: github.com/Privatris/AgentLeak
- name: AgentLeakTester
  role: Instantiated with mode=DetectionMode.HYBRID (or other modes); exposes check()
    and batch_check() to measure per-check latency and leak/confidence outcomes
  repo: github.com/Privatris/AgentLeak
- name: GPT-4o-mini
  role: Target model used in the reproduction workflow (openai/gpt-4o-mini) when running
    benchmark traces through the configured detection mode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/access.2026.3704541
    title: 'AgentLeak: A Benchmark for Internal-Channel Privacy Leakage in Multi-Agent
      LLM Systems'
  source_content_hash: sha256:ed7eae45a63efe81149b8bce1c4ea20955a65ad239ff63341aa43641771b9ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AgentLeakTester
  - GPT-4o
  - GPT-4o-mini
  - agentleak
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multi-agent-detection-mode-configuration

## Summary

Configure and evaluate the AgentLeak three-tier detection pipeline's DetectionMode (FAST, STANDARD, HYBRID, LLM_ONLY) for privacy-leakage checks across multi-agent channels, matching each mode's latency profile (~10ms, ~100ms, ~200ms, ~2s) to its intended deployment (e.g., HYBRID for production, LLM_ONLY for audit).

## When to use

Use when deploying or benchmarking AgentLeakTester against multi-agent LLM traces and you must pick a DetectionMode whose latency budget (FAST ~10ms to LLM_ONLY ~2s) and role (HYBRID = production) match the evaluation channel.

## When NOT to use

- You only need a single-model external-output audit without latency profiling — the full mode sweep over FAST/STANDARD/HYBRID/LLM_ONLY is unnecessary overhead
- Your system has no multi-agent internal channels (C2 inter-agent, C5 memory, C3-C4 tools) — the mode designations were characterized for internal-channel leakage in multi-agent systems
- Latency tolerance is far below ~10ms per check — even the FAST mode's profile makes it unsuitable for sub-10ms inline filtering

## Inputs

- agentleak package installed locally (pip install -e .) with OPENROUTER_API_KEY in a .env file
- AgentLeak benchmark scenarios (1,000 scenarios across healthcare, finance, legal, corporate domains; channels C1 output, C2 inter-agent, C5 memory)
- DetectionMode enum values: FAST, STANDARD, HYBRID, LLM_ONLY
- Vault dict of sensitive values (e.g., {"ssn": "123-45-6789"}) and candidate output strings per channel

## Outputs

- Per-mode measured latency values compared to the reported profile (FAST ~10ms, STANDARD ~100ms, HYBRID ~200ms, LLM_ONLY ~2s)
- Detection results with result.leaked and result.confidence fields per check
- detection_mode_results.csv compiling per-mode latency and detection outcomes
- A mode-to-role assignment (HYBRID = production, LLM_ONLY = audit)

## How to apply

Instantiate AgentLeakTester from the agentleak package with an explicit DetectionMode (e.g., DetectionMode.HYBRID) after installing with `pip install -e .` and storing the OpenRouter API key in a .env file. For each of the four modes (FAST, STANDARD, HYBRID, LLM_ONLY), run the benchmark scenarios through both the single-check (tester.check with a vault dict, output string, and channel label such as C1) and batch_check interfaces, recording per-check latency. Compare measured latency against the reported profile (FAST ~10ms, STANDARD ~100ms, HYBRID ~200ms, LLM_ONLY ~2s) within reasonable tolerance, and assign modes by designation: HYBRID is designated for production use and LLM_ONLY for audit. Optionally re-run the CLI with a defense enabled (`python -m agentleak run --defense D1`) to capture detection behavior under defense, and compile per-mode latency and detection outcomes into detection_mode_results.csv.

## Related tools

- **agentleak** (Core framework SDK providing the tester, DetectionMode enum, and CLI (`python -m agentleak run`) used to sweep and evaluate detection modes) — github.com/Privatris/AgentLeak
- **AgentLeakTester** (Instantiated with mode=DetectionMode.HYBRID (or other modes); exposes check() and batch_check() to measure per-check latency and leak/confidence outcomes) — github.com/Privatris/AgentLeak
- **GPT-4o-mini** (Target model used in the reproduction workflow (openai/gpt-4o-mini) when running benchmark traces through the configured detection mode)

## Examples

```
from agentleak import AgentLeakTester, DetectionMode; tester = AgentLeakTester(mode=DetectionMode.HYBRID); result = tester.check(vault={"ssn": "123-45-6789"}, output="The SSN is 123-45-6789", channel="C1")
```

## Evaluation signals

- Each mode's measured per-check latency matches its reported value (FAST ~10ms, STANDARD ~100ms, HYBRID ~200ms, LLM_ONLY ~2s) within reasonable tolerance
- Detection designation is consistent: HYBRID is the production configuration and LLM_ONLY is used for audit
- tester.check returns a result object with `leaked` and `confidence` fields for each vault/output/channel triple
- CLI runs with a defense (e.g., `--defense D1`) complete and their detection outcomes are recorded alongside latency in detection_mode_results.csv
- Test suite passes (`pytest tests/ -v`) after installation and .env configuration

## Limitations

- Latency values are approximate profile figures (~10ms to ~2s per check); LLM_ONLY at ~2s per check is impractical for inline production filtering
- Detection behavior depends on model availability via OpenRouter and a correctly configured OPENROUTER_API_KEY in .env
- The pipeline was characterized on AgentLeak's 1,000 scenarios and 7 channels; latency/role designations may not transfer to other scenario distributions

## Evidence

- [methods] HYBRID mode designated for production: [evidence span withheld — anchor status: not_found]
- [readme] README usage snippet instantiating HYBRID mode: "tester = AgentLeakTester(mode=DetectionMode.HYBRID)"
- [other] Defense-enabled CLI run from workflow: "python -m agentleak run --defense D1         # With defense"
