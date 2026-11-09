---
name: in-context-learning-prompt-design
description: Use when building an LLM QA or reasoning workflow that must detect, via
  generated token sequences, when the model needs external knowledge so that constrained
  decoding over a knowledge index can be activated.
when_to_use_negative:
- When the model's parametric knowledge already covers the dataset (e.g., Mintaka,
  WebQSP where LLM-only reached 82.0% and 80.5% accuracy) — adding the Fact-command
  loop can lower accuracy.
- When answers must come from a retrieval pipeline with an external retriever or auxiliary
  model rather than constrained generation over a pre-built prefix tree.
- When the knowledge base cannot be verbalized into tokenized textual facts for prefix-tree
  indexing.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: Qwen2.5-3B
  role: Target LLM that receives the ICL system prompt and whose output token stream
    is scanned for the Fact command
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed prefix-tree index consulted during the constrained-decoding segments
    triggered by Fact-command detection
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:e7642738c5a2a82b401f3ce0a26f7ce3b440b6bd3993c9314f8f2590513ab697
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# in-context-learning-prompt-design

## Summary

Designs an in-context learning (ICL) system prompt that instructs an LLM (e.g., Qwen2.5-3B) to reason step by step and to invoke a Fact command when external knowledge is required, serving as the trigger that switches the decoder between normal and constrained generation in ReFactX.

## When to use

Use when building an LLM QA or reasoning workflow that must detect, via generated token sequences, when the model needs external knowledge so that constrained decoding over a knowledge index can be activated.

## When NOT to use

- When the model's parametric knowledge already covers the dataset (e.g., Mintaka, WebQSP where LLM-only reached 82.0% and 80.5% accuracy) — adding the Fact-command loop can lower accuracy.
- When answers must come from a retrieval pipeline with an external retriever or auxiliary model rather than constrained generation over a pre-built prefix tree.
- When the knowledge base cannot be verbalized into tokenized textual facts for prefix-tree indexing.

## Inputs

- Base instruction-tuned LLM (e.g., Qwen2.5-3B, Qwen2.5-7B, Phi-4)
- User question / QA task prompt
- Prefix-tree fact index (in-memory 31k-facts KB or disk-backed PostgreSQL Wikidata index)

## Outputs

- ICL system prompt defining step-by-step reasoning and Fact-command invocation
- Mixed-mode generation stream alternating normal and constrained decoding segments
- Fact-grounded final answer or an explicit 'I don't know' response

## How to apply

Compose a system prompt that (1) tells the LLM to reason step by step, and (2) instructs it to invoke the Fact command whenever external facts are required instead of relying on parametric knowledge. During generation, continuously scan output tokens for the Fact-command token sequence; detection is the runtime switch that activates constrained generation over the prefix-tree index (in-memory from a 31k-facts KB, or disk-backed PostgreSQL holding up to 800M verbalized Wikidata facts). After each complete fact is emitted, revert to normal unconstrained decoding so multi-step reasoning can continue, and include termination rules so the model either answers from grounded facts or outputs 'I don't know'. The prompt wording matters because the command must be emitted as an exact, detectable token sequence for the control loop to trigger.

## Related tools

- **Qwen2.5-3B** (Target LLM that receives the ICL system prompt and whose output token stream is scanned for the Fact command) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed prefix-tree index consulted during the constrained-decoding segments triggered by Fact-command detection) — https://github.com/rpo19/ReFactX

## Examples

```
from refactx import pipeline; pipeline.run(model='Qwen2.5-3B', index='31k_in_memory', system_prompt='Reason step by step; invoke the Fact command when external facts are required.')
```

## Evaluation signals

- Every constrained segment in the output decodes to an existing KB fact in the prefix tree.
- Fact-command token sequence is reliably detected at inference time, toggling constrained generation on and off within one run.
- Demo run terminates with either a fact-grounded answer or an explicit 'I don't know' response (or the max-token budget, e.g., 4000 tokens).
- Precision of fact-grounded answers exceeds the LLM-only baseline in LLM-as-a-Judge evaluation (ReFactX always outperforms LLM-only in precision).
- Constrained-generation overhead stays ~1.3% (303.89s vs 300.14s for 4000 tokens), confirming the control loop adds negligible latency.

## Limitations

- Constrained generation only narrows the vocabulary to tokens forming an existing KB fact; the final token choice always remains with the LLM's probability estimates.
- If the prompt fails to elicit a clean Fact-command token sequence, the normal/constrained switching control loop will not trigger.
- Accuracy can drop relative to LLM-only answering on datasets well covered by parametric knowledge (Mintaka, WebQSP).
- The Bank dataset proved especially challenging, with precision up to only 51.3% and accuracy up to 42.8%.

## Evidence

- [intro] The LLM's generation of the token sequence corresponding to the Fact command is detected at inference time and triggers activation of constrained generation: [evidence span withheld — anchor status: not_found]
