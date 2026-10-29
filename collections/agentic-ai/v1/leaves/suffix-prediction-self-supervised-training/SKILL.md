---
name: suffix-prediction-self-supervised-training
description: 'Use when adapting a retrieval-augmented LLM at inference time to a specialized
  domain: retrieved passages are available, no labeled training data exists, and the
  model must self-supervisedly learn to predict retrieved content.'
when_to_use_negative:
- The task is full domain retraining with labeled data rather than label-free adaptation
  at inference time
- Retrieved passages are too short to yield meaningful prefix-suffix context for suffix
  prediction
- You need to reproduce the no-segmentation ablation baseline, which by design skips
  prefix-suffix splitting and performs next-token prediction on the entire passage
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TTARAG
  role: Framework implementing the test-time adaptation loop with prefix-suffix suffix-prediction
    training (tta_module.py, RAGModel)
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation model whose parameters are adapted via suffix prediction at test
    time
- name: parallel_evaluation.py
  role: Entry point that runs generation with --do_tta, learning rate, and adaptation-pair
    settings
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Hardware used to run adaptation and generation over 2,706 CRAG queries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:b1789c55c44f89b2a449dec93b5463175415b82acc9102240c012d4b0d1fee9d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Llama-3.1-8B-Instruct
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - parallel_evaluation.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# suffix-prediction-self-supervised-training

## Summary

Generates self-supervised adaptation signals for test-time RAG adaptation by splitting each retrieved passage into prefix-suffix pairs and training the language model to predict the suffix from the prefix, enabling dynamic parameter updates to the target domain without labeled data.

## When to use

Use when adapting a retrieval-augmented LLM at inference time to a specialized domain: retrieved passages are available, no labeled training data exists, and the model must self-supervisedly learn to predict retrieved content.

## When NOT to use

- The task is full domain retraining with labeled data rather than label-free adaptation at inference time
- Retrieved passages are too short to yield meaningful prefix-suffix context for suffix prediction
- You need to reproduce the no-segmentation ablation baseline, which by design skips prefix-suffix splitting and performs next-token prediction on the entire passage

## Inputs

- Top-20 retrieved sentences per query (e.g. BM25 top-5 passages reranked by MiniLM-v6 sentence selection on CRAG)
- Pre-trained instruction-tuned LLM weights (e.g. models/Llama-3.1-8B-Instruct)
- TTA configuration (learning rate, max_adapt_pairs, minimum passage length threshold)

## Outputs

- Adapted model parameters used to generate the final response
- Model predictions saved under predictions/ and summaries under results/<dataset>/
- Per-query accuracy scores (LLM-as-a-judge semantic-equivalence, e.g. Qwen2.5-72B-Instruct)

## How to apply

First apply length filtering, discarding passages shorter than a configured minimum length threshold so that each passage has enough context for meaningful adaptation. Split each surviving retrieved passage into prefix-suffix pairs using the paper's two-tier splitting strategy, then compute the loss on the suffix prediction task with prefix tokens masked during label preparation. Accumulate gradients over two steps, clip to a maximum norm threshold, and update parameters with the AdamW optimizer; use a learning rate between 1e-6 and 1e-5 (e.g. 1e-5) and 3-5 adaptation pairs (e.g. 3), which the paper reports as optimal and to which performance is not highly sensitive. Reset model parameters to their original pre-trained state before each adaptation iteration to ensure a clean starting point, then generate the final response using the adapted parameters. This prefix-suffix splitting is the mechanism that matters: the 'wo seg' ablation that predicts whole passages without splitting consistently underperforms on CRAG (30.8/30.1/25.0 vs 31.9/30.5/25.7 for Llama-3.1-8b-it, Llama-2-7b-chat, ChatGLM-3-6b).

## Related tools

- **TTARAG** (Framework implementing the test-time adaptation loop with prefix-suffix suffix-prediction training (tta_module.py, RAGModel)) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation model whose parameters are adapted via suffix prediction at test time)
- **parallel_evaluation.py** (Entry point that runs generation with --do_tta, learning rate, and adaptation-pair settings) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Hardware used to run adaptation and generation over 2,706 CRAG queries)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3
```

## Evaluation signals

- Adapted-parameter answers on CRAG overall reach ~31.9 accuracy for Llama-3.1-8B-Instruct (vs ~30.8 for the wo-seg whole-passage ablation), with positive deltas across all three models
- Learning rate and adaptation pairs fall in the reported optimal ranges (1e-6 to 1e-5; 3-5 pairs; e.g. 1e-5 and 3)
- Model parameters are verified to be reset to the pre-trained state before each adaptation iteration
- Prefix tokens are confirmed masked in label preparation, with loss computed only on suffix tokens
- Per-query processing time stays in the reported range (1.75s for 1 pair up to 2.60s for 5 pairs)

## Limitations

- Performance is not highly sensitive to the number of adaptation pairs and learning rate, so tuning gains are limited
- Passages shorter than the minimum length threshold cannot be used for adaptation
- Processing time per query grows with the number of adaptation pairs (1.75s for 1 pair to 2.60s for 5 pairs)

## Evidence
