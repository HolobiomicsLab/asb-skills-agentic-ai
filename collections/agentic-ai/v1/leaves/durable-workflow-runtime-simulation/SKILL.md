---
name: durable-workflow-runtime-simulation
description: Use when you must test crash, retry, or duplicate-effect behavior of
  a durable workflow/recovery harness and attribute each outcome to a named kill point
  and declared invariant, using a naive negative-control adapter.
when_to_use_negative:
- The goal is to benchmark model capability (e.g. pass@k on a task set) rather than
  to test harness crash/retry/duplicate-effect behavior — use an evaluation-comparison
  protocol instead
- The workflow under test has no external commitments or durable transitions, so no
  post-commitment kill points or duplicate-effect invariants exist to evaluate
- You need results from an actually executed system rather than a simulated reconstruction
  — the finding here is described per the sweep mechanism, not as a reported executed
  result
tools:
- name: git
  role: Clone the engineering-reliable-coding-agents repository to read the recovery-fault-injection
    protocol and Chapters 9–10 material
  repo: https://github.com/sjarmak/engineering-reliable-coding-agents
- name: Python 3
  role: Implement the simulated durable-step runtime with typed event stream, injectable
    crash points, naive retry adapter, and the five-kill-point sweep
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:99363b319046b6ae0f8d20d9005290d189d9286ab8b210ec071141133e223976
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# durable-workflow-runtime-simulation

## Summary

A fault-injection simulation skill from the recovery-fault-injection protocol (Chapters 9–10 of Engineering Reliable Coding Agents): build a simulated durable-step runtime with a typed event stream and injectable crash points, then sweep a named set of kill points to tie duplicate-effect outcomes to invariants rather than to generic flakiness.

## When to use

Use when you must test crash, retry, or duplicate-effect behavior of a durable workflow/recovery harness and attribute each outcome to a named kill point and declared invariant, using a naive negative-control adapter.

## When NOT to use

- The goal is to benchmark model capability (e.g. pass@k on a task set) rather than to test harness crash/retry/duplicate-effect behavior — use an evaluation-comparison protocol instead
- The workflow under test has no external commitments or durable transitions, so no post-commitment kill points or duplicate-effect invariants exist to evaluate
- You need results from an actually executed system rather than a simulated reconstruction — the finding here is described per the sweep mechanism, not as a reported executed result

## Inputs

- protocols/recovery-fault-injection.md from the engineering-reliable-coding-agents repository (named kill points, declared invariants, pass condition)
- Chapters 9–10 material on persistent agent state, durable workflows, and idempotent retries
- Python 3 implementation of a durable-step runtime with typed events (dispatch, external commitment, acknowledgment, durable transition) and injectable crash points

## Outputs

- recovery_fault_injection_results file containing per-kill-point typed event records with invariant outcomes for the plain run and the naive-retry run
- Per-kill-point verdicts against the declared invariants (no duplicate external effect, at-most-once commitment)

## How to apply

Read protocols/recovery-fault-injection.md and the Chapters 9–10 material from the cloned engineering-reliable-coding-agents repository to extract the named kill points, declared invariants (e.g. no duplicate external effect, at-most-once commitment), and pass condition. Implement in Python 3 a simulated durable-step runtime with a typed event stream covering dispatch, external commitment, acknowledgment, and durable transition, with injectable crash points between each stage. Attach a naive retry adapter that resumes from the last durable state without idempotency keys, so retried logical work can re-issue external effects — this is the designed negative control. Run the fault-injection sweep at each of the five kill points (before dispatch; after dispatch before external commitment; after commitment before acknowledgment; after acknowledgment before durable transition; after transition), recording the typed event sequence and invariant outcomes for both the plain run and the naive-retry run. The rationale is the monograph's system-level claim: once a run can outlive its worker and publish durable effects, reliability is a property of the surrounding system, so failures must be localized to a specific kill point and invariant rather than reported as unexplained model failures.

## Related tools

- **git** (Clone the engineering-reliable-coding-agents repository to read the recovery-fault-injection protocol and Chapters 9–10 material) — https://github.com/sjarmak/engineering-reliable-coding-agents
- **Python 3** (Implement the simulated durable-step runtime with typed event stream, injectable crash points, naive retry adapter, and the five-kill-point sweep)

## Examples

```
git clone https://github.com/sjarmak/engineering-reliable-coding-agents.git
```

## Evaluation signals

- The output file exists and contains an entry for each of the five named kill points
- The naive retry adapter shows duplicate effects at post-commitment kill points (the designed negative-control failure exposing violation of the no-duplicates invariant)
- No duplicate effect is observed at the pre-dispatch kill point (crash before any external commitment)
- Each recorded outcome is tied to a named kill point and a declared invariant, not to an unattributed error
- Typed event sequences (dispatch, external commitment, acknowledgment, durable transition) are captured for both plain and naive-retry runs

## Limitations

- The negative-control expectation is grounded in the described sweep mechanism rather than a reported executed result
- Findings from this skill characterize the harness/system layer, not the model; improvements at one layer may still fail to propagate to end-to-end outcomes
- The naive retry adapter is deliberately non-idempotent (no idempotency keys, no reconciliation or deduplication), so it is a designed failure control, not a production recovery strategy

## Evidence

- [methods] Five named kill points define the sweep: [evidence span withheld — anchor status: not_found]
- [results] Naive negative control violates the no-duplicates invariant after external commitment: [evidence span withheld — anchor status: not_found]
- [results] Validation distinguishes post-commitment duplicates from pre-dispatch correctness: [evidence span withheld — anchor status: not_found]
