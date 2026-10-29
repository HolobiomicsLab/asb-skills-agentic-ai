---
name: typed-event-stream-modeling
description: Use when you must test crash, retry, or duplicate-effect behavior of
  a workflow that makes external commitments, and you need each failure tied to a
  named kill point and a declared invariant (e.g. no duplicate external effect, at-most-once
  commitment).
when_to_use_negative:
- The workflow makes no external commitments or durable effects, so there is no duplicate-effect
  invariant to violate and the sweep has nothing meaningful to expose
- You are evaluating raw model capability on static benchmarks (e.g. code generation
  scores) rather than system-level recovery behavior around the model
- The recovery adapter already implements reconciliation/idempotency keys and you
  only need functional testing — the negative-control comparison is then unnecessary
  overhead
tools:
- name: Python 3
  role: Implement the simulated durable-step runtime with the typed event stream and
    injectable crash points, and run the five-kill-point sweep
- name: git
  role: Clone the engineering-reliable-coding-agents repository to read the recovery-fault-injection
    protocol and Chapters 9–10 material
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents.git
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:75eecbf2179e6bedb93100fa2e19ccb27ec02bb3176bb38bddc3bf177f6edf09
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents.git
  tools:
  - Python
  - Python 3
  - git
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Typed Event Stream Modeling for Recovery Fault Injection

## Summary

Models a durable-step workflow as a typed event stream (dispatch, external commitment, acknowledgment, durable transition) with injectable crash points, so that recovery fault injection can tie a duplicate-effect failure to a named kill point and a declared invariant rather than to an unexplained crash.

## When to use

Use when you must test crash, retry, or duplicate-effect behavior of a workflow that makes external commitments, and you need each failure tied to a named kill point and a declared invariant (e.g. no duplicate external effect, at-most-once commitment).

## When NOT to use

- The workflow makes no external commitments or durable effects, so there is no duplicate-effect invariant to violate and the sweep has nothing meaningful to expose
- You are evaluating raw model capability on static benchmarks (e.g. code generation scores) rather than system-level recovery behavior around the model
- The recovery adapter already implements reconciliation/idempotency keys and you only need functional testing — the negative-control comparison is then unnecessary overhead

## Inputs

- Protocol document naming the kill points, invariants, and pass condition (protocols/recovery-fault-injection.md)
- Workflow definition with at least one external call that produces a durable external effect
- Adapter configuration: system recovery path plus a naive retry adapter without idempotency keys or deduplication

## Outputs

- Typed event sequences (dispatch, external commitment, acknowledgment, durable transition) per kill point per run
- Per-kill-point records of invariant outcomes (e.g. no duplicate external effect, at-most-once commitment) for plain and naive-retry runs
- Results file (e.g. recovery_fault_injection_results.) with one entry per named kill point

## How to apply

Read the protocol (e.g. protocols/recovery-fault-injection.md and Chapters 9–10 of the engineering-reliable-coding-agents monograph) to extract the named kill points, declared invariants, and pass condition before writing any code. Implement in Python 3 a simulated durable-step runtime whose event types cover dispatch, external commitment, acknowledgment, and durable transition, with injectable crash points between each stage. Attach both the system-under-test recovery path and a naive negative-control adapter that restarts from the first step without reconciliation or idempotency keys, so retried logical work can re-issue external effects — this control is designed to expose the failure. Run the sweep at each named kill point (before dispatch, after dispatch before commitment, after commitment before acknowledgment, after acknowledgment before durable transition, after transition), recording the typed event sequence per run. Evaluate the declared invariants for plain and naive-retry runs and write per-kill-point event records with invariant outcomes to the results file.

## Related tools

- **Python 3** (Implement the simulated durable-step runtime with the typed event stream and injectable crash points, and run the five-kill-point sweep)
- **git** (Clone the engineering-reliable-coding-agents repository to read the recovery-fault-injection protocol and Chapters 9–10 material) — https://github.com/sjarmak/engineering-reliable-coding-agents.git

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && python3 run_recovery_fault_injection.py --kill-points 5 --output recovery_fault_injection_results.
```

## Evaluation signals

- The output file exists and contains an entry for each of the five named kill points
- The naive retry adapter shows duplicate external effects at every kill point after an external commitment
- The naive retry adapter shows no duplicate effect at the pre-dispatch kill point (no commitment has been made yet)
- Each kill-point entry records the full typed event sequence (dispatch, external commitment, acknowledgment, durable transition)
- Invariants evaluated are the ones declared in the protocol (e.g. no duplicate external effect, at-most-once commitment), not ad-hoc checks

## Limitations

- The naive negative-control outcome is the protocol's designed/expected failure (the control must violate the no-duplicates invariant at post-commitment kill points) — per the described sweep mechanism, not a reported executed result
- The runtime is a simulation of durable-step behavior in Python 3; results are only as faithful as the modeled event stages and crash-point placement
- Evidence strength varies by topic and empirical results depend on workload and system configuration, so sweep conclusions do not automatically transfer to other harnesses

## Evidence
