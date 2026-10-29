---
name: contrastive-feature-alignment-training
description: Use when training a cross-modal fusion head that must align pooled textual
  representations with prototype-refined visual features via a scaled dot-product
  alignment score and cross-entropy supervision.
when_to_use_negative:
- Input is raw meme images/text without precomputed Long-CLIP visual tokens and CoT
  text embeddings — encode and generate multi-CoT first
- The task requires training the vision or text encoder itself; this skill assumes
  frozen Long-CLIP encoders and trains only the fusion head
- Standard CLIP is being considered for the text branch, since it cannot handle the
  ~150-word CoT sequences (struggles beyond 20 tokens)
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Long-CLIP
  role: Frozen visual and text encoder supplying visual tokens H and text embeddings
    T_emb; extends context to 248 tokens for the ~150-word CoT texts
- name: AdamW
  role: Optimizer (lr 5e-5, weight decay 0.01) with cosine learning rate scheduler
    used to train the fusion module
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:59688ef3b5aa1625ffcf71d9579ddc5576f6fe0ee98df222877196a44857dde4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AdamW
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# contrastive-feature-alignment-training

## Summary

A training procedure for a prototype-guided/context-guided decoder fusion module in which refined visual features S and enhanced hateful/benign text embeddings are aligned via a temperature-scaled dot-product alignment score and supervised with cross-entropy, optimized with AdamW (lr 5e-5, weight decay 0.01, cosine scheduler).

## When to use

Use when training a cross-modal fusion head that must align pooled textual representations with prototype-refined visual features via a scaled dot-product alignment score and cross-entropy supervision.

## When NOT to use

- Input is raw meme images/text without precomputed Long-CLIP visual tokens and CoT text embeddings — encode and generate multi-CoT first
- The task requires training the vision or text encoder itself; this skill assumes frozen Long-CLIP encoders and trains only the fusion head
- Standard CLIP is being considered for the text branch, since it cannot handle the ~150-word CoT sequences (struggles beyond 20 tokens)

## Inputs

- Visual tokens H in R^{N x M x D} from the frozen Long-CLIP visual encoder
- Long-CLIP text embeddings T_emb for hateful and benign CoT texts of each Hateful Memes example
- Hateful Memes training labels (hateful / non-hateful)

## Outputs

- Classification logits from the tau-scaled alignment score (Eq. 7)
- Trained learnable prototypes Pr, attention matrices W_a/W_b/W_c/W_v, and temperature tau
- Validation accuracy and AUROC on Test Seen / Test Unseen

## How to apply

After context-guided attention produces enriched hateful and benign text features Th_embE and Tb_embE from visual prototype queries against Long-CLIP text embeddings, concatenate the average-pooled hateful and benign text features and compute the scaled dot product with the refined visual features S, multiplying by a learnable temperature parameter tau (Eq. 7) to obtain classification logits. Supervise these logits with cross-entropy against ground-truth hateful/non-hateful labels from the Hateful Memes dataset. Train with AdamW at learning rate 5e-5, weight decay 0.01, and a cosine learning rate scheduler, using batch size 8 with gradient accumulation every 2 steps. Keep the Long-CLIP visual and text encoders frozen so only the prototype, attention matrices (W_a, W_b, W_c, W_v), and tau are learned. Evaluate with accuracy and AUROC, expecting roughly 75.88/79.25 on Test Seen and 75.57/79.40 on Test Unseen.

## Related tools

- **Long-CLIP** (Frozen visual and text encoder supplying visual tokens H and text embeddings T_emb; extends context to 248 tokens for the ~150-word CoT texts)
- **AdamW** (Optimizer (lr 5e-5, weight decay 0.01) with cosine learning rate scheduler used to train the fusion module)

## Examples

```
python train/train.py --dataset hateful_memes --lr 5e-5 --weight_decay 0.01 --scheduler cosine --batch_size 8 --grad_accum 2
```

## Evaluation signals

- Validation accuracy on Hateful Memes Test Seen approaches the reported 75.88% (AUROC ~79.25)
- Test Unseen metrics land near 75.57 accuracy / 79.40 AUROC
- Removing Long-CLIP should noticeably reduce ACC and AUROC — if not, the frozen encoders are likely not wired correctly
- Learnable temperature tau produces logits whose cross-entropy decreases stably under AdamW with cosine scheduler at batch size 8 with gradient accumulation every 2 steps
- MultiOFF transfer reaches ~70.0 accuracy and 63.8 F1 when evaluated on that dataset

## Limitations

- The authors' future work targets shortening the reasoning chains while maintaining performance, implying long multi-CoT chains are a cost/efficiency burden
- The public repository (BGWH123/EVL-MCoT) currently ships only the train/ pipeline; models/, configs/, scripts/, pretrained weights, and usage instructions are listed as upcoming, so exact reproducibility is limited
- Performance depends on the chosen MLLM (LLaVA-1.5-7B or GPT-4) for generating the multi-CoT texts (n=3, m=3 setting)

## Evidence

- [methods] alignment score with learnable temperature: [evidence span withheld — anchor status: not_found]
- [methods] optimizer settings: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [methods] context-guided attention: "we employ a context-guided attention layer, where the visual prototype features Pr ∼ serve as the queries Q, while the text features T emb serve as both the keys K and values V"
- [results] headline metrics: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [results] ablation signal: [evidence span withheld — anchor status: not_found]
- [readme] repo status: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
