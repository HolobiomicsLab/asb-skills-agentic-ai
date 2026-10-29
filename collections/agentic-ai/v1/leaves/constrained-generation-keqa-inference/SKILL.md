---
name: constrained-generation-keqa-inference
description: Use when answering KG-based QA questions (e.g., 2WikiMH, Mintaka, WebQSP,
  Bank) with an LLM whose parametric knowledge is insufficient or hallucination-prone,
  and you want fact-grounded generation against a large KB like Wikidata's 800M facts
  without a pipeline retriever.
when_to_use_negative:
- The questions are already covered by the model's parametric knowledge — on Mintaka
  and WebQSP, LLM-only reached 82.0% and 80.5% accuracy and ReFactX performed worse
  in accuracy
- The knowledge base is tiny or toy-scale where a 31k-facts in-memory index (try_refactx
  notebook) suffices and the disk-backed 800M-fact PostgreSQL infrastructure is unnecessary
  overhead
- You need retrieval-style free-text evidence or provenance rather than token-constrained
  emission of existing KB facts
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: Qwen2.5-72B
  role: Base LLM performing ICL-prompted generation under prefix-tree constraints;
    achieved P=85.2% and A=78.9 with LLM-as-a-Judge, and 92.5% accuracy on the benchmark
    set
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Alternative base LLM evaluated with constrained generation (P=77.8%, C=70.6%)
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-7B
  role: Smaller base LLM for constrained-generation QA, achieving 73.0% accuracy over
    the large KB
  repo: https://github.com/rpo19/ReFactX
- name: Phi-4
  role: Base LLM evaluated with constrained generation; maintains consistent precision
    across all answer types
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: Model used in the latency/throughput experiment with PostgreSQL on the same
    machine and key-value caching enabled
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Serves the disk-backed Wikidata prefix-tree index during constrained generation
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: GPU used to run inference with key-value caching enabled in the timing experiments
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:5d66aa00fb8663945131e5a33e53754d419f12c1b5525d1abe6f56ca595330ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Llama3.3-70B
  - Nvidia Tesla T4
  - Phi-4
  - PostgreSQL
  - Qwen2.5-3B
  - Qwen2.5-72B
  - Qwen2.5-7B
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# constrained-generation-keqa-inference

## Summary

Runs ReFactX-style knowledge-enhanced QA, where an LLM answers questions by invoking a Fact command via In-Context Learning and then generating facts through constrained decoding over a pre-built prefix-tree index of verbalized Wikidata triples, enabling external-knowledge grounding without retrievers or auxiliary models.

## When to use

Use when answering KG-based QA questions (e.g., 2WikiMH, Mintaka, WebQSP, Bank) with an LLM whose parametric knowledge is insufficient or hallucination-prone, and you want fact-grounded generation against a large KB like Wikidata's 800M facts without a pipeline retriever.

## When NOT to use

- The questions are already covered by the model's parametric knowledge — on Mintaka and WebQSP, LLM-only reached 82.0% and 80.5% accuracy and ReFactX performed worse in accuracy
- The knowledge base is tiny or toy-scale where a 31k-facts in-memory index (try_refactx notebook) suffices and the disk-backed 800M-fact PostgreSQL infrastructure is unnecessary overhead
- You need retrieval-style free-text evidence or provenance rather than token-constrained emission of existing KB facts

## Inputs

- Natural-language QA questions from benchmarks such as 2WikiMH, Mintaka, WebQSP, or Bank
- A pre-built prefix-tree index of verbalized Wikidata facts (disk-backed, ~800M facts) served via PostgreSQL
- An instruction-tuned LLM (e.g., Llama3.3-70B, Phi-4, Qwen2.5-3B/7B/72B) served with key-value caching

## Outputs

- Generated answers containing KB-grounded facts emitted along valid prefix-tree paths
- Accuracy and precision scores under Exact Match and LLM-as-a-Judge evaluation per model/dataset pair
- Throughput/latency measurements from throughput.py (e.g., 4000-token generation time, ~1.3% constrained-generation overhead)

## How to apply

Build or load a disk-backed prefix-tree index in which Wikidata KG triples are verbalized in textual facts, tokenized, and indexed (the paper's index holds up to 800M facts and adds only ~1% latency, served through PostgreSQL with key-value caching on an Nvidia Tesla T4). Install the package in a virtualenv/conda environment via `pip install refactx[torch]` and configure the `.env` file from `env-sample.txt`. At inference time, instruct the LLM via In-Context Learning to invoke the Fact command; once the command is recognized, constrained generation is activated so the model can only emit token sequences that form an existing fact in the prefix tree, after which decoding reverts to normal. Decision point: constrained generation only narrows the vocabulary to tokens forming existing KB facts — the final token choice always remains with the LLM's probability estimates, so quality still depends on the base model (e.g., Qwen2.5-7B vs Qwen2.5-72B). Score outputs with Exact Match and LLM-as-a-Judge accuracy and precision, comparing against unconstrained (LLM-only) runs as the parametric-knowledge baseline.

## Related tools

- **Qwen2.5-72B** (Base LLM performing ICL-prompted generation under prefix-tree constraints; achieved P=85.2% and A=78.9 with LLM-as-a-Judge, and 92.5% accuracy on the benchmark set) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Alternative base LLM evaluated with constrained generation (P=77.8%, C=70.6%)) — https://github.com/rpo19/ReFactX
- **Qwen2.5-7B** (Smaller base LLM for constrained-generation QA, achieving 73.0% accuracy over the large KB) — https://github.com/rpo19/ReFactX
- **Phi-4** (Base LLM evaluated with constrained generation; maintains consistent precision across all answer types) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (Model used in the latency/throughput experiment with PostgreSQL on the same machine and key-value caching enabled) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Serves the disk-backed Wikidata prefix-tree index during constrained generation) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (GPU used to run inference with key-value caching enabled in the timing experiments) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- ReFactX answers must always outperform LLM-only baselines in precision under LLM-as-a-Judge evaluation (an invariant observed across all model/dataset pairs)
- Reproduce the paper's headline figures: Qwen2.5-72B judge P=85.2% / A=78.9; accuracy of 73.0% with Qwen2.5-7B and 92.5% with the larger Qwen2.5 model
- On 2WikiMH, constrained-generation runs should improve both accuracy and precision by more than 20% relative to LLM-only models (e.g., 92.5%/96.4% vs 45.0%/45.2%)
- Constrained generation overhead should stay ~1.3%: generating 4000 tokens takes ~303.89s vs ~300.14s unconstrained (300.14 → 303.89 measured with Qwen2.5-3B, PostgreSQL, key-value caching, one T4)
- Verify the prefix-tree index loads and serves valid fact paths per PrefixTree.md before running eval.py

## Limitations

- ReFactX performs worse than LLM-only in accuracy on Mintaka and WebQSP, where parametric knowledge covers questions to a large extent
- The Bank dataset proved especially challenging, with precision only up to 51.3% and accuracy up to 42.8%
- Constrained generation narrows the vocabulary to existing KB facts but leaves the final token choice to the LLM, so hallucinated fact selection is still possible
- Disk-backed index requires the PostgreSQL setup described in PrefixTree.md; skipped configuration limits you to the 31k-facts in-memory notebook demo

## Evidence

- [intro] ICL-prompted constrained decoding: "the LLM is instructed via In-Context Learning (ICL) [7] to invoke the Fact command when external facts are required. Once the command is recognized, constrained generation is activated"
- [intro] Index scale and latency: "a disk-backed prefix tree holds up to 800M facts and adds only ∼1 % latency."
