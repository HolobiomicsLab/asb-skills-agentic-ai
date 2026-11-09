---
name: medical-domain-accuracy-benchmark-evaluation
description: Use when evaluating or reproducing RAG system accuracy on biomedical
  QA benchmarks (BioASQ, PubMedQA) to quantify whether test-time adaptation improves
  accuracy over baseline RAG under domain distribution shift.
when_to_use_negative:
- The evaluation target is the CRAG benchmark (2,706 web/KG question-answer pairs
  with mock search APIs) rather than the medical BioASQ/PubMedQA datasets, which use
  different retrieval corpora and scoring conventions.
- Only baseline retrieval-augmented prompting (naive-RAG, CoT, ICL) is being compared
  without any test-time adaptation, so the TTA parameter reset, prefix-suffix adaptation,
  and AdamW update steps are unnecessary.
- Gold reference answers are unavailable, since the LLM-judge scoring requires comparing
  predictions against reference answers for semantic equivalence.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3303
- http://edamontology.org/topic_3474
tools:
- name: Pyserini
  role: Implements the BM25 retriever that fetches the top 5 relevant passages per
    question from the biomedical corpora
- name: TTARAG
  role: 'Test-time adaptation method: resets parameters, splits retrieved passages
    into prefix-suffix pairs, adapts on 3 pairs at lr 1e-5, and generates answers
    with adapted parameters'
  repo: https://github.com/sunxin000/TTARAG
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge that determines semantic equivalence between model predictions
    and reference answers to produce accuracy scores
- name: Llama-3.1-8B-Instruct
  role: Backbone generation model for both baseline and TTARAG evaluation runs
- name: NVIDIA A100 GPU
  role: Hardware used to run the LLM generation and adaptation experiments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:98dbea2bb6a0f29eff2a040f618285d6a66a99d5bbac9ff4cb516eaa6cbf0f13
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3303
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# medical-domain-accuracy-benchmark-evaluation

## Summary

Reproduce and benchmark RAG accuracy on the BioASQ (500 QA pairs) and PubMedQA (1,000 QA pairs) medical datasets, comparing naive-RAG, Chain-of-Thought, In-Context Learning, and TTARAG across Llama-3.1-8B-Instruct, Llama-2-7b-chat, and ChatGLM-3-6b backbones, with answers scored by an LLM-as-a-judge.

## When to use

Use when evaluating or reproducing RAG system accuracy on biomedical QA benchmarks (BioASQ, PubMedQA) to quantify whether test-time adaptation improves accuracy over baseline RAG under domain distribution shift.

## When NOT to use

- The evaluation target is the CRAG benchmark (2,706 web/KG question-answer pairs with mock search APIs) rather than the medical BioASQ/PubMedQA datasets, which use different retrieval corpora and scoring conventions.
- Only baseline retrieval-augmented prompting (naive-RAG, CoT, ICL) is being compared without any test-time adaptation, so the TTA parameter reset, prefix-suffix adaptation, and AdamW update steps are unnecessary.
- Gold reference answers are unavailable, since the LLM-judge scoring requires comparing predictions against reference answers for semantic equivalence.

## Inputs

- BioASQ dataset (500 expert-curated biomedical question-answer pairs)
- PubMedQA dataset (1,000 biomedical research question-answer pairs, qiaojin/PubMedQA pqa_labeled split)
- Biomedical corpora for BM25 retrieval (PubMed abstracts, biomedical articles)
- Backbone model weights: Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b
- TTAConfig settings: learning_rate 1e-5, max_adapt_pairs 3

## Outputs

- medical_domain_accuracy.csv with BioASQ and PubMedQA accuracy (%) for naive-rag, +CoT, +ICL, and TTARAG per model plus deltas vs naive-rag
- Per-question model predictions saved under predictions/ and summaries under results/<dataset>/
- LLM-judge semantic-equivalence verdicts (Qwen2.5-72B-Instruct) for each prediction vs gold answer

## How to apply

Load BioASQ (500 expert-curated QA pairs) and PubMedQA (1,000 biomedical research QA pairs, pqa_labeled split), retrieving the top 5 relevant passages per question with a BM25 retriever implemented in Pyserini over PubMed abstracts and biomedical articles. Run naive-RAG, Chain-of-Thought, and In-Context Learning baselines for each backbone (Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b). For TTARAG, reset parameters to the pre-trained state, split retrieved passages into prefix-suffix pairs, filter short passages, adapt on 3 adaptation pairs with learning rate 1e-5 using AdamW with gradient accumulation over 2 steps and norm clipping, then generate answers with the adapted parameters (theta'). Score all predictions against gold answers with Qwen2.5-72B-Instruct as an LLM-as-a-judge to determine semantic equivalence. Compile accuracy (%) per method per model plus deltas vs naive-RAG into a single table (mirroring the paper's Table 1 medical columns) saved as medical_domain_accuracy.csv.

## Related tools

- **Pyserini** (Implements the BM25 retriever that fetches the top 5 relevant passages per question from the biomedical corpora)
- **TTARAG** (Test-time adaptation method: resets parameters, splits retrieved passages into prefix-suffix pairs, adapts on 3 pairs at lr 1e-5, and generates answers with adapted parameters) — https://github.com/sunxin000/TTARAG
- **Qwen2.5-72B-Instruct** (LLM-as-a-judge that determines semantic equivalence between model predictions and reference answers to produce accuracy scores)
- **Llama-3.1-8B-Instruct** (Backbone generation model for both baseline and TTARAG evaluation runs)
- **NVIDIA A100 GPU** (Hardware used to run the LLM generation and adaptation experiments)

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3
```

## Evaluation signals

- TTARAG accuracy exceeds naive-RAG accuracy on both BioASQ and PubMedQA for all three backbones (Llama-3.1-8B-Instruct, Llama-2-7b-chat, ChatGLM-3-6b).
- Reproduced gains match reported values, e.g. Llama-3.1-8b-it: 75.0 vs 55.6 on BioASQ (+19.4%) and +10.8% on PubMedQA.
- The LLM-judge shows 98% agreement with human annotators on a random sample of 100 examples, so sampled judge verdicts should be spot-checked against that rate.
- The output CSV contains all four method rows (naive-rag, +CoT, +ICL, TTARAG) per model with numeric accuracy percentages and deltas vs naive-rag, mirroring the paper's Table 1 medical columns.
- Adaptation hyperparameters fall in the validated regime: learning rate between 1e-6 and 1e-5 and 3-5 adaptation pairs, which the authors report yields optimal performance.

## Limitations

- TTARAG processing time per query ranges from 1.75s (1 adaptation pair) to 2.60s (5 pairs), though this is still lower than Chain-of-Thought's 4.32s.
- Performance is not sensitive to the number of adaptation pairs or learning rate, so small hyperparameter changes are unlikely to rescue a failed reproduction.
- The article notes no quantitative metrics were reported in some sections supporting the improvement claim over naive-rag, so reproducers must rely on the Table 1 medical-domain values.
- Each adaptation iteration requires resetting model parameters to their original pre-trained state to ensure a clean starting point; omitting this corrupts per-question comparisons.
- CRAG auto-evaluation uses correct (1 point), missing (0 points), incorrect (-1 point) scoring, which differs from the medical datasets' LLM-judge semantic-equivalence accuracy.

## Evidence

- [methods] PubMedQA dataset size: [evidence span withheld — anchor status: not_found]
- [methods] LLM-as-a-judge evaluation: [evidence span withheld — anchor status: not_found]
- [methods] Judge-human agreement: [evidence span withheld — anchor status: not_found]
- [methods] Optimal TTA hyperparameter range: [evidence span withheld — anchor status: not_found]
- [intro] TTARAG best-results claim: [evidence span withheld — anchor status: not_found]
