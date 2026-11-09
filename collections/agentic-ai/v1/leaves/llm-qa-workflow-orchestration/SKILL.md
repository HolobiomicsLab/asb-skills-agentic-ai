---
name: llm-qa-workflow-orchestration
description: Use when an LLM must answer knowledge-base questions by invoking an external-fact
  command at inference time and you need to implement the detect-command → constrain-decoding
  → resume-reasoning control loop.
when_to_use_negative:
- Questions already well covered by the LLM's parametric knowledge — on Mintaka and
  WebQSP, LLM-only models reached 82.0% and 80.5% accuracy and ReFactX performed worse
- You need a retriever/pipeline architecture with a separate retrieval model — ReFactX
  is explicitly positioned as an alternative to pipeline-based solutions with no external
  retrievers
- Extremely constrained compute where even the ~1% disk-index latency and 1.3% constrained-generation
  time overhead are unacceptable
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: Qwen2.5-3B
  role: Base LLM driven via ICL to invoke the Fact command and generate facts under
    constrained decoding; also evaluated at 7B/70B/72B scales
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed prefix-tree fact index queried during constrained generation,
    adding only ~1% latency with up to 800M facts
  repo: https://github.com/rpo19/ReFactX
- name: refactx[torch]
  role: Python package that installs the QA loop, in-memory prefix tree, and constrained-generation
    machinery
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:f1417e6eeb1d8c0559f4fe2ea5d5a5b13f4df57c6d94c82279bc4d25f4154378
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  - refactx[torch]
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# llm-qa-workflow-orchestration

## Summary

Orchestrates an ICL-driven LLM QA control loop in which generation of the Fact-command token sequence at inference time switches decoding into constrained generation over a pre-built prefix-tree fact index, then reverts to normal decoding so multi-step reasoning can continue until a grounded answer, an explicit 'I don't know', or the token budget is reached.

## When to use

Use when an LLM must answer knowledge-base questions by invoking an external-fact command at inference time and you need to implement the detect-command → constrain-decoding → resume-reasoning control loop.

## When NOT to use

- Questions already well covered by the LLM's parametric knowledge — on Mintaka and WebQSP, LLM-only models reached 82.0% and 80.5% accuracy and ReFactX performed worse
- You need a retriever/pipeline architecture with a separate retrieval model — ReFactX is explicitly positioned as an alternative to pipeline-based solutions with no external retrievers
- Extremely constrained compute where even the ~1% disk-index latency and 1.3% constrained-generation time overhead are unacceptable

## Inputs

- Natural-language QA questions (e.g. from Mintaka, WebQSP, 2WikiMH, Bank datasets)
- Tokenized fact index: in-memory prefix tree from a 31k-facts KB, or disk-backed PostgreSQL prefix tree over ~800M verbalized Wikidata triples
- System prompt with ICL instructions to invoke the Fact command
- Configuration .env file (cp env-sample.txt .env, then edit) for index/model settings

## Outputs

- Fact-grounded natural-language answers
- Explicit 'I don't know' responses when no supporting fact exists in the KB
- Constrained generation segments that each decode to an existing KB fact in the prefix tree

## How to apply

Compose a system prompt instructing the LLM (e.g. Qwen2.5-3B) via In-Context Learning to reason step by step and invoke the Fact command when external knowledge is required. Load the tokenized fact index through the refactx[torch] package — an in-memory prefix tree from a 31k-facts KB for quick trials, or a disk-backed PostgreSQL index for large KBs (up to 800M Wikidata facts, adding only ~1% latency). During generation, scan output for the Fact-command token sequence; on detection, activate constrained generation so only tokens along valid prefix-tree paths (i.e. existing KB facts) can be emitted, while leaving the final token choice to the LLM's probability estimates. After each complete fact is emitted, revert decoding to normal (unconstrained) mode so reasoning can continue. Apply termination rules: produce the final answer, output 'I don't know' when no supporting fact is found, or stop at the max-token budget (4000 tokens in the throughput experiments, ~303.89s constrained vs 300.14s unconstrained, a 1.3% overhead).

## Related tools

- **Qwen2.5-3B** (Base LLM driven via ICL to invoke the Fact command and generate facts under constrained decoding; also evaluated at 7B/70B/72B scales) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed prefix-tree fact index queried during constrained generation, adding only ~1% latency with up to 800M facts) — https://github.com/rpo19/ReFactX
- **refactx[torch]** (Python package that installs the QA loop, in-memory prefix tree, and constrained-generation machinery) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every constrained segment must decode to an existing KB fact in the prefix tree
- The full loop must terminate with either a fact-grounded answer or an explicit 'I don't know' response
- Precision of fact-grounded answers should exceed the LLM-only baseline — ReFactX always outperforms LLM-only models in precision under LLM-as-a-Judge evaluation (e.g. P=85.2% with Qwen2.5-72B as judge)
- Constrained generation time overhead should stay near the reported ~1.3% (303.89s vs 300.14s for 4000 tokens)
- On 2WikiMH, accuracy and precision should improve by more than 20% over LLM-only models

## Limitations

- Underperforms LLM-only models in accuracy on Mintaka and WebQSP, where parametric knowledge already covers most questions
- The Bank dataset proved especially challenging, with precision up to only 51.3% and accuracy up to 42.8%
- Requires building the tokenized prefix-tree index over the KB in advance (via PrefixTree.md procedure for Wikidata)
- Knowledge gaps and hallucinations persist when no supporting fact exists, mitigated only by the 'I don't know' fallback

## Evidence
