---
name: biomedical-qa-bm25-retrieval
description: Use when benchmarking RAG systems on biomedical question answering, e.g.
  when inputs are BioASQ or PubMedQA question-answer pairs and you need domain-specific
  accuracy comparisons against baselines.
when_to_use_negative:
- The task is general/web-domain RAG rather than biomedical QA — the paper's general
  benchmark is CRAG (2,706 QA pairs with Brave Search API retrieval and MiniLM-v6
  sentence reranking), which is a different pipeline.
- No gold reference answers are available and LLM-as-a-judge semantic equivalence
  cannot be computed against gold answers.
- Only a single small model or extremely limited compute is available and running
  three backbones plus a 72B judge on an A100 GPU is infeasible.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: Pyserini
  role: Implements the BM25 retriever fetching the top 5 relevant passages per question
    from biomedical corpora
  repo: https://github.com/sunxin000/TTARAG
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge determining semantic equivalence between model predictions
    and reference answers (98% agreement with human annotators)
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Test-time adaptation module (models/tta_module.py) applied to the RAG model
    with learning rate 1e-5 and max_adapt_pairs 3
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-3.1-8B-Instruct
  role: Generation backbone evaluated in the medical-domain accuracy comparison
  repo: https://github.com/sunxin000/TTARAG
- name: Llama-2-7b-chat
  role: Generation backbone evaluated in the medical-domain accuracy comparison
- name: ChatGLM-3-6b
  role: Generation backbone evaluated in the medical-domain accuracy comparison
- name: NVIDIA A100 GPU
  role: GPU used to run the evaluation pipeline (e.g. 2,706 queries on one A100 in
    the paper's benchmark runs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:df0e58502a8f221b31fb5998b444227a03cf8096ef8de5ede049d899837d3550
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - ChatGLM-3-6b
  - Llama-2-7b-chat
  - Llama-3.1-8B-Instruct
  - NVIDIA A100 GPU
  - Pyserini
  - Qwen2.5-72B-Instruct
  - TTARAG
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# biomedical-qa-bm25-retrieval

## Summary

Reproduce the medical-domain RAG evaluation on BioASQ (500 QA pairs) and PubMedQA (1,000 QA pairs) by retrieving top-5 passages with a Pyserini BM25 retriever, running naive-RAG/CoT/ICL/TTARAG pipelines over biomedical corpora, and scoring answers with Qwen2.5-72B-Instruct as an LLM-as-a-judge to build the paper's Table 1 medical accuracy columns.

## When to use

Use when benchmarking RAG systems on biomedical question answering, e.g. when inputs are BioASQ or PubMedQA question-answer pairs and you need domain-specific accuracy comparisons against baselines.

## When NOT to use

- The task is general/web-domain RAG rather than biomedical QA — the paper's general benchmark is CRAG (2,706 QA pairs with Brave Search API retrieval and MiniLM-v6 sentence reranking), which is a different pipeline.
- No gold reference answers are available and LLM-as-a-judge semantic equivalence cannot be computed against gold answers.
- Only a single small model or extremely limited compute is available and running three backbones plus a 72B judge on an A100 GPU is infeasible.

## Inputs

- BioASQ dataset (500 expert-curated biomedical QA pairs)
- PubMedQA dataset (1,000 biomedical research QA pairs, loaded via qiaojin/PubMedQA pqa_labeled)
- Biomedical retrieval corpora (PubMed abstracts and biomedical articles)
- Backbone model weights (Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b)
- NVIDIA A100 GPU

## Outputs

- medical_domain_accuracy.csv with BioASQ and PubMedQA accuracy (%) for naive-rag, +CoT, +ICL, and TTARAG per backbone, plus deltas vs naive-RAG
- Per-question predictions (saved under predictions/) and summaries (under results/<dataset>/)
- LLM-judge semantic equivalence verdicts per question

## How to apply

Load BioASQ (500 expert-curated QA pairs from biomedical literature) and PubMedQA (1,000 biomedical research QA pairs), and retrieve the top 5 relevant passages per question using a BM25 retriever implemented in Pyserini over PubMed abstracts and biomedical articles respectively. Run naive-RAG, Chain-of-Thought, and In-Context Learning baselines for each backbone (Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b). For TTARAG, reset parameters to the pre-trained state, split retrieved passages into prefix-suffix pairs, adapt on 3 pairs with learning rate 1e-5 (the 1e-6 to 1e-5 range is optimal), accumulate gradients over 2 steps with norm clipping, and update with AdamW; generate answers with the adapted parameters theta'. Score predictions against gold answers with Qwen2.5-72B-Instruct judging semantic equivalence, which showed 98% agreement with human annotators on 100 examples. Compile accuracy (%) per method and backbone plus deltas vs naive-RAG into a single table (e.g. medical_domain_accuracy.csv) mirroring the paper's Table 1 medical columns.

## Related tools

- **Pyserini** (Implements the BM25 retriever fetching the top 5 relevant passages per question from biomedical corpora) — https://github.com/sunxin000/TTARAG
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge determining semantic equivalence between model predictions and reference answers (98% agreement with human annotators)) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Test-time adaptation module (models/tta_module.py) applied to the RAG model with learning rate 1e-5 and max_adapt_pairs 3) — https://github.com/sunxin000/TTARAG
- **Llama-3.1-8B-Instruct** (Generation backbone evaluated in the medical-domain accuracy comparison) — https://github.com/sunxin000/TTARAG
- **Llama-2-7b-chat** (Generation backbone evaluated in the medical-domain accuracy comparison)
- **ChatGLM-3-6b** (Generation backbone evaluated in the medical-domain accuracy comparison)
- **NVIDIA A100 GPU** (GPU used to run the evaluation pipeline (e.g. 2,706 queries on one A100 in the paper's benchmark runs))

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- TTARAG accuracy exceeds naive-RAG on both BioASQ and PubMedQA for all three backbones, matching reported gains such as BioASQ +19.4% and PubMedQA +10.8% for Llama-3.1-8b-it (75.0 vs 55.6 on BioASQ)
- The compiled table mirrors the paper's Table 1 medical columns with naive-rag, +CoT, +ICL, and TTARAG rows per backbone and delta columns vs naive-rag
- Retrieval returns exactly 5 passages per question from the correct corpus (PubMed abstracts for PubMedQA, biomedical articles for BioASQ)
- LLM-judge agreement with human annotators reproduces at ~98% on a random sample of 100 examples
- TTA adaptation runs with the documented hyperparameters: learning rate 1e-5, 3 adaptation pairs, gradient accumulation over 2 steps with norm clipping, AdamW updates, and parameters reset to pre-trained state per iteration

## Limitations

- TTARAG performance is not sensitive to the number of adaptation pairs and learning rate, but 1e-6 to 1e-5 learning rates with 3-5 adaptation pairs are recommended for the best effectiveness/efficiency balance
- Average processing time per query ranges from 1.75s (1-pair) to 2.60s (5-pair), so adaptation adds latency over naive-RAG though it is lower than CoT's 4.32s
- Evaluation depends on an LLM-as-a-judge rather than exact match, so scoring requires access to a large judge model (Qwen2.5-72B-Instruct)
- The README requires local model weights (e.g. models/Llama-3.1-8B-Instruct and models/allMiniLM) to be placed or overridden via --model_name, and PubMedQA is loaded from the qiaojin/PubMedQA pqa_labeled split via the dataset adapter

## Evidence

- [methods] BM25 retrieval top-5: "we employed a standard BM25 retriever implemented using the Pyserini library to fetch the top 5 relevant passages"
- [methods] PubMedQA dataset size: [evidence span withheld — anchor status: not_found]
- [methods] BioASQ dataset size: "BioASQ [28], comprising 500 expert-curated question-answer pairs from the biomedical literature"
- [methods] LLM-as-a-judge: [evidence span withheld — anchor status: not_found]
- [methods] TTA hyperparameters: [evidence span withheld — anchor status: not_found]
- [methods] Judge-human agreement: [evidence span withheld — anchor status: not_found]
- [methods] Adaptation loop details: "gradients are accumulated over two steps and clipped to a maximum norm threshold. The AdamW optimizer then updates model parameters"
- [readme] README TTA invocation: "python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3"
