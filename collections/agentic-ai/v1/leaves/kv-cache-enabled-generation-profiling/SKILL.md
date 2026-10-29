---
name: kv-cache-enabled-generation-profiling
description: Use when you need to measure the wall-clock time overhead that constrained
  generation (e.g., a prefix-tree index restricting valid token sequences) adds over
  normal LLM decoding under identical hardware and settings.
when_to_use_negative:
- You want to measure answer quality (precision/accuracy) of constrained generation
  rather than latency — use the eval.py QA-benchmark workflow instead
- Your generation budget is very short (a few tokens), where per-call index-lookup
  latency may dominate and the ~1.3% overhead measured over 4000 tokens will not be
  representative
- You are profiling retrieval-augmented pipelines with separate retriever models —
  this skill isolates only the constrained-decoding overhead with no external retriever
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: throughput.py
  role: Script that times generation of a fixed token budget (max-tokens 4001) with
    or without --unconstrained-generation and writes timings to out.json
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: Language model used for both constrained and unconstrained decoding runs
- name: PostgreSQL
  role: Disk-backed store for the prefix-tree index consulted during constrained decoding
- name: Nvidia Tesla T4
  role: Single GPU on which both timed runs are executed to keep hardware constant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:c7a52695d4dddd973d4dfa81ea5f3435d2b8b582923625a85dc5e242025db8e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Nvidia Tesla T4
  - PostgreSQL
  - Qwen2.5-3B
  - throughput.py
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# kv-cache-enabled-generation-profiling

## Summary

Profiles LLM generation throughput to quantify the latency overhead of KB-guided constrained decoding versus unconstrained decoding, with key-value caching enabled. Used to show that a disk-backed prefix-tree constraint adds only ~1.3% overhead over 4000 generated tokens.

## When to use

Use when you need to measure the wall-clock time overhead that constrained generation (e.g., a prefix-tree index restricting valid token sequences) adds over normal LLM decoding under identical hardware and settings.

## When NOT to use

- You want to measure answer quality (precision/accuracy) of constrained generation rather than latency — use the eval.py QA-benchmark workflow instead
- Your generation budget is very short (a few tokens), where per-call index-lookup latency may dominate and the ~1.3% overhead measured over 4000 tokens will not be representative
- You are profiling retrieval-augmented pipelines with separate retriever models — this skill isolates only the constrained-decoding overhead with no external retriever

## Inputs

- Language model to profile (Qwen2.5-3B in the reported experiment)
- PostgreSQL-backed prefix-tree index over tokenized facts
- throughput.py script with --model, --index, --max-tokens, --output, and optional --unconstrained-generation flags
- .env configuration file prepared from env-sample.txt

## Outputs

- Total wall-clock generation time (seconds) for the unconstrained baseline run (e.g., 300.14 s for 4000 tokens)
- Total wall-clock generation time (seconds) for the constrained run (e.g., 303.89 s for 4000 tokens)
- Relative overhead percentage computed as (constrained_time - unconstrained_time) / unconstrained_time (e.g., 1.3%), written to out.json

## How to apply

Set up the ReFactX environment (virtualenv/conda, pip install refactx[torch]) and prepare the .env configuration from env-sample.txt pointing to the PostgreSQL-backed prefix-tree index. Run the unconstrained baseline with throughput.py (python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json --unconstrained-generation) using Qwen2.5-3B with key-value caching enabled on one Nvidia Tesla T4, recording total wall-clock time for 4000 tokens (reported baseline: 300.14 s). Re-run without the --unconstrained-generation flag so generation is constrained by the PostgreSQL-backed prefix tree to only token sequences forming existing facts, recording the total time (reported: 303.89 s). Compute the relative overhead as constrained_time/unconstrained_time - 1: (303.89-300.14)/300.14 = 1.3%. Both runs must use identical model, index, max-tokens, caching, and hardware so the only differing factor is the constraint mechanism.

## Related tools

- **throughput.py** (Script that times generation of a fixed token budget (max-tokens 4001) with or without --unconstrained-generation and writes timings to out.json) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (Language model used for both constrained and unconstrained decoding runs)
- **PostgreSQL** (Disk-backed store for the prefix-tree index consulted during constrained decoding)
- **Nvidia Tesla T4** (Single GPU on which both timed runs are executed to keep hardware constant)

## Examples

```
python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json --unconstrained-generation
```

## Evaluation signals

- Unconstrained baseline time for 4000 tokens lands near the reported 300.14 s on a Tesla T4 with Qwen2.5-3B and KV caching enabled
- Constrained time lands near the reported 303.89 s, i.e., relative overhead close to 1.3%
- Both runs use identical settings except the constraint flag (same model, index, max-tokens 4001, key-value caching, single GPU)
- Both timings and the computed overhead are persisted to out.json
- The constrained run generates only token sequences forming existing facts in the prefix-tree index (no invalid continuations)

## Limitations

- The ~1.3% overhead is measured with a 3B model, key-value caching enabled, PostgreSQL on the same machine, and one Tesla T4; other models, hardware, or index placement may shift the number
- Overhead is characterized over a long generation (4000 tokens); the prefix-tree adds only ~1% latency for the 800M-fact index, but short generations are not profiled
- Results depend on the prefix-tree index being pre-built and accessible via PostgreSQL; index construction time is not part of this measurement

## Evidence
