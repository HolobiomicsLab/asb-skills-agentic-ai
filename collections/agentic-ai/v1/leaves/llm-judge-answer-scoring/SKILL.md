---
name: llm-judge-answer-scoring
description: Use when a batch of free-text model answers must be scored against gold
  reference answers for QA benchmarks like CRAG, PubMedQA, or BioASQ, where string
  matching is insufficient and semantic-equivalence judging is needed.
when_to_use_negative:
- The task has deterministic exact-match answers where rule-based matching (as in
  CRAG's local_evaluation.py) suffices without LLM judging
- A smaller/cheaper model is substituted for the judge without first verifying judge-human
  agreement, since the 98% agreement figure is specific to Qwen2.5-72B-Instruct
- Inputs are retrieval passages or model checkpoints rather than generated answer
  strings paired with gold references
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge that determines semantic equivalence between model predictions
    and reference answers
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Source of generated predictions under predictions/ and evaluation prompts;
    local_evaluation.py implements scoring and debugging
  repo: https://github.com/sunxin000/TTARAG
- name: CRAG
  role: Benchmark whose evaluation protocol (perfect/acceptable/missing/incorrect
    ratings with rule-based matching plus LLM assessment) is adopted for the judge
  repo: https://github.com/facebookresearch/CRAG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:12387bb870d854b64eaa832ee26eedb5ef65d849ba23ebd4be74965ef86c6348
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - CRAG
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# llm-judge-answer-scoring

## Summary

Score generated RAG answers against gold references using an LLM-as-a-judge (Qwen2.5-72B-Instruct) that judges semantic equivalence, adopted from the CRAG evaluation protocol. This yields automated accuracy metrics without per-example human annotation.

## When to use

Use when a batch of free-text model answers must be scored against gold reference answers for QA benchmarks like CRAG, PubMedQA, or BioASQ, where string matching is insufficient and semantic-equivalence judging is needed.

## When NOT to use

- The task has deterministic exact-match answers where rule-based matching (as in CRAG's local_evaluation.py) suffices without LLM judging
- A smaller/cheaper model is substituted for the judge without first verifying judge-human agreement, since the 98% agreement figure is specific to Qwen2.5-72B-Instruct
- Inputs are retrieval passages or model checkpoints rather than generated answer strings paired with gold references

## Inputs

- Generated answer strings per query (e.g., predictions/ files from parallel_evaluation.py)
- Gold reference answers from the benchmark (CRAG 2,706 QA pairs, PubMedQA pqa_labeled, BioASQ)
- Original benchmark questions/query IDs
- Evaluation prompt templates from the prompts/ directory

## Outputs

- Per-query judge verdicts (semantically equivalent or not)
- Aggregate accuracy per experimental configuration
- Summary files under results/<dataset>/ (e.g., sweep tables over learning rate and number of adaptation pairs)

## How to apply

Take the generated answers and their gold references and present each (question, prediction, reference) triple to Qwen2.5-72B-Instruct, which judges whether the prediction is semantically equivalent to the reference; this protocol is adopted from the CRAG evaluation, which rates responses as perfect, acceptable, missing, or incorrect and assigns scores of correct (1), missing (0), and incorrect (-1). Run the judge over all predictions saved under predictions/ and aggregate accuracy per experimental configuration into summaries under results/<dataset>/. Validate the judge itself by measuring agreement with human annotators on a random sample of 100 examples; this protocol reached 98% agreement. When a hyperparameter sweep (e.g., learning rates 1e-6 to 1e-5 with 1-5 adaptation pairs) is being compared, apply the identical judge to every configuration so accuracy values are comparable across the sweep table and plots.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge that determines semantic equivalence between model predictions and reference answers) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Source of generated predictions under predictions/ and evaluation prompts; local_evaluation.py implements scoring and debugging) — https://github.com/sunxin000/TTARAG
- **CRAG** (Benchmark whose evaluation protocol (perfect/acceptable/missing/incorrect ratings with rule-based matching plus LLM assessment) is adopted for the judge) — https://github.com/facebookresearch/CRAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3
```

## Evaluation signals

- Judge-human agreement on a random sample of 100 examples should be high (the paper reports 98% for Qwen2.5-72B-Instruct)
- Scores must fall in the CRAG scheme: correct (1), missing (0), incorrect (-1)
- Every generated prediction must receive a verdict — no unanswered/query IDs missing from the summary under results/<dataset>/
- Applying the same judge across all sweep configurations (learning rates 1e-6 to 1e-5, 1-5 adaptation pairs) should reproduce near-flat accuracy curves with TTARAG above the naive-rag baseline in almost all settings

## Limitations

- The 98% agreement with human annotators was measured on only a random sample of 100 examples, so judge reliability on rare answer types is unverified
- Judge behavior is tied to the specific model (Qwen2.5-72B-Instruct) and prompt templates (prompts/); swapping either requires re-validating agreement
- LLM assessment supplements but does not replace CRAG's rule-based matching, which is also part of the automatic evaluation

## Evidence

- [methods] employ an LLM-as-a-judge (Qwen2.5-72B-Instruct) to determine semantic equivalence between model predictions and reference answers: [evidence span withheld — anchor status: not_found]
- [methods] the LLM-judger demonstrates 98% agreement with human annotators on a random sample of 100 examples: [evidence span withheld — anchor status: not_found]
- [readme] Automatic evaluation employs rule-based matching and LLM assessment to check answer correctness. It will assign three scores: correct (1 point), missing (0 points), and incorrect (-1 point).: [evidence span withheld — anchor status: not_found]
- [methods] Score all generated answers against gold references using the LLM-as-a-judge (Qwen2.5-72B-Instruct) semantic-equivalence protocol adopted from the CRAG evaluation.: [evidence span withheld — anchor status: not_found]
