---
name: distributed-systems-invariant-verification
description: 'Use when testing crash, retry, or duplicate-effect behavior of a durable
  workflow: you have named kill points and declared invariants and need a fault result
  tied to each kill point, including a negative control that exposes designed failures.'
when_to_use_negative:
- The question concerns model capability or evaluation scores rather than crash/recovery
  behavior of durable workflow state — use the minimum reliability pass or evaluation-comparison
  protocol instead.
- No declared invariants or named kill points exist to test against; fault injection
  without a predeclared pass condition cannot produce a defensible fault result.
- The workflow never issues external effects or durable transitions, so post-commitment
  duplicate-effect invariants are vacuous.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Python 3
  role: Implement the simulated durable-step runtime, typed event stream, injectable
    crash points, and naive retry adapter, and run the kill-point sweep
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: git
  role: Clone the engineering-reliable-coding-agents repository to obtain the protocol
    file and Chapters 9–10 material
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:8b3b523332f863a65191179ac4ac66d1709eb9b87063ebbe5a83cb54ef650549
  source_papers:
  - doi: 10.48550/arxiv.2608.13867
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sjarmak/engineering-reliable-coding-agents
  tools:
  - Python
  - Python 3
  - git
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# distributed-systems-invariant-verification

## Summary

A fault-injection method for verifying durable-workflow invariants: build a simulated durable-step runtime with a typed event stream and injectable crash points, then sweep named kill points to check that declared invariants (e.g. no duplicate external effect, at-most-once commitment) hold for a correct adapter and are deliberately violated by a naive negative control.

## When to use

Use when testing crash, retry, or duplicate-effect behavior of a durable workflow: you have named kill points and declared invariants and need a fault result tied to each kill point, including a negative control that exposes designed failures.

## When NOT to use

- The question concerns model capability or evaluation scores rather than crash/recovery behavior of durable workflow state — use the minimum reliability pass or evaluation-comparison protocol instead.
- No declared invariants or named kill points exist to test against; fault injection without a predeclared pass condition cannot produce a defensible fault result.
- The workflow never issues external effects or durable transitions, so post-commitment duplicate-effect invariants are vacuous.

## Inputs

- Recovery fault-injection protocol document (e.g. protocols/recovery-fault-injection.md) naming kill points, invariants, and pass condition
- Monograph chapters on durable workflows and idempotent retries (Chapters 9–10) from the engineering-reliable-coding-agents repository
- Python 3 harness implementation with injectable crash points between event-stream stages

## Outputs

- Per-kill-point event records with invariant outcomes written to recovery_fault_injection_results (one entry per named kill point for plain and naive-retry runs)
- Typed event sequences (dispatch, external commitment, acknowledgment, durable transition) per kill point
- Invariant verdicts: no-duplicate-external-effect and at-most-once-commitment outcomes per adapter per kill point

## How to apply

Read the protocol file (e.g. protocols/recovery-fault-injection.md) and the relevant monograph chapters (here Chapters 9–10 on persistent agent state, durable workflows, and idempotent retries) to extract named kill points, declared invariants, and the pass condition. Implement in Python 3 a simulated durable-step runtime with a typed event stream covering dispatch, external commitment, acknowledgment, and durable transition, with injectable crash points between each stage. Attach the adapter under test — plus a naive negative-control retry adapter that resumes from the last durable state without idempotency keys, reconciliation, or deduplication. Run the sweep at each of the five kill points: before dispatch, after dispatch before external commitment, after commitment before acknowledgment, after acknowledgment before durable transition, and after transition; for each, record the typed event sequence and evaluate the declared invariants for both runs. The expected design outcome is that the naive control violates the no-duplicates invariant at every kill point after an external commitment, while the pre-dispatch kill point shows no duplicate effect.

## Related tools

- **Python 3** (Implement the simulated durable-step runtime, typed event stream, injectable crash points, and naive retry adapter, and run the kill-point sweep) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **git** (Clone the engineering-reliable-coding-agents repository to obtain the protocol file and Chapters 9–10 material) — https://github.com/sjarmak/engineering-reliable-coding-agents

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git && python3 recovery_fault_injection.py --protocol protocols/recovery-fault-injection.md --out recovery_fault_injection_results
```

## Evaluation signals

- The output file exists and contains an entry for each of the five named kill points (before dispatch, after dispatch before external commitment, after commitment before acknowledgment, after acknowledgment before durable transition, after transition).
- The naive retry adapter shows duplicate external effects at every post-commitment kill point and no duplicate effect at the pre-dispatch kill point.
- Each recorded entry includes the typed event sequence for both the plain run and the naive-retry run.
- Invariant evaluations (no-duplicate-external-effect, at-most-once-commitment) are reported per kill point against the protocol's declared pass condition.
- Kill points and invariants match those named in protocols/recovery-fault-injection.md and the Chapters 9–10 material, not invented ones.

## Limitations

- The finding describes the protocol's designed sweep mechanism and expected naive-control failure, not a reported executed result; running the sweep is required to produce actual measurements.
- The naive negative control is deliberately unreconciled (no idempotency keys, reconciliation, or deduplication), so its violations characterize the designed failure rather than production adapter behavior.
- Empirical results remain dependent on workload and system configuration; fault outcomes are tied to specific kill points and do not generalize across harness revisions without re-running the sweep.

## Evidence

- [methods] Workflow defines five kill points and typed event stream: [evidence span withheld — anchor status: not_found]
- [methods] Validation criterion for the sweep output: [evidence span withheld — anchor status: not_found]
- [intro] Reliability becomes a system property once runs mutate durable state: [evidence span withheld — anchor status: not_found]
