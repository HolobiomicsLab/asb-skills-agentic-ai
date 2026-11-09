---
name: cross-modal-attention-fusion
description: Use when fine-grained visual-textual alignment is needed for meme or
  vision-language classification, especially when standard CLIP encoders fail on long
  CoT texts (>20 tokens, ~150-word sequences).
when_to_use_negative:
- Inputs are short (<20 token) captions where standard CLIP suffices — the prototype/context
  decoders target long ~150-word CoT sequences
- The task requires a single-chain reasoning baseline rather than multi-CoT visual-textual
  fusion
- Only unimodal text or only unimodal image data is available, so cross-attention
  fusion over visual tokens and text embeddings cannot be formed
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Long-CLIP
  role: Frozen visual and text encoders producing visual tokens H and text embeddings
    T_emb; extends context to 248 tokens for long CoT texts
- name: AdamW
  role: Optimizer for training the fusion module (lr 5e-5, weight decay 0.01, cosine
    scheduler)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:318537e9181aae3952f57227adde9aa9cff8590b47538fa20804045a15ad7ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AdamW
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-modal-attention-fusion

## Summary

A prototype-guided and context-guided decoder fusion framework that aligns frozen Long-CLIP visual tokens with hateful/benign CoT text embeddings via learnable prototypes and cross-attention, overcoming the shallow feature fusion of prior two-stream models.

## When to use

Use when fine-grained visual-textual alignment is needed for meme or vision-language classification, especially when standard CLIP encoders fail on long CoT texts (>20 tokens, ~150-word sequences).

## When NOT to use

- Inputs are short (<20 token) captions where standard CLIP suffices — the prototype/context decoders target long ~150-word CoT sequences
- The task requires a single-chain reasoning baseline rather than multi-CoT visual-textual fusion
- Only unimodal text or only unimodal image data is available, so cross-attention fusion over visual tokens and text embeddings cannot be formed

## Inputs

- Visual tokens H in R^{N x M x D} from the frozen Long-CLIP visual encoder for each Hateful Memes example
- Text embeddings T_emb from the frozen Long-CLIP text encoder for hateful and benign CoT texts (n=3, m=3 chains)
- Ground-truth hateful/non-hateful labels for cross-entropy supervision

## Outputs

- Refined visual features S from prototype-guided fusion
- Enriched hateful and benign text representations Th_embE and Tb_embE
- Classification logits from the temperature-scaled alignment score (Eq. 7)

## How to apply

Load visual tokens H in R^{N x M x D} and text embeddings T_emb from frozen Long-CLIP encoders. In the prototype-guided decoder, initialize P learnable prototypes Pr in R^{P x D} and run cross-attention with prototypes as queries and visual tokens as keys/values, Softmax(Pr H^T / sqrt(D))H, followed by layer normalization and a residual connection (Eq. 2). Derive refined visual features S via attention-based feature fusion over prototype tokens using tanh-softmax attention weights A computed with trainable matrices W_a, W_b, W_c, W_v (Eqs. 3-5). In the context-guided decoder, treat the refined visual prototype features Pr~ as queries and text features T_emb as keys and values (Eq. 6) so the text encoder captures visual-contextual information, yielding enriched hateful and benign text representations Th_embE and Tb_embE. Finally compute classification logits by concatenating average-pooled hateful and benign text features, taking the scaled dot product with S, and multiplying by a learnable temperature tau (Eq. 7), supervised with cross-entropy.

## Related tools

- **Long-CLIP** (Frozen visual and text encoders producing visual tokens H and text embeddings T_emb; extends context to 248 tokens for long CoT texts)
- **AdamW** (Optimizer for training the fusion module (lr 5e-5, weight decay 0.01, cosine scheduler))

## Examples

```
git clone https://github.com/BGWH123/EVL-MCoT && python train/train.py --optimizer adamw --lr 5e-5 --weight_decay 0.01 --scheduler cosine --batch_size 8 --grad_accum 2
```

## Evaluation signals

- Accuracy near the reported 75.88% and AUROC near 79.25 on Hateful Memes Test Seen (75.57%/79.40 on Test Unseen)
- Removing Long-CLIP from the pipeline significantly reduces ACC and AUROC, confirming the encoder-fusion dependency
- Attention outputs follow the expected forms: Softmax(Pr H^T / sqrt(D))H in the prototype-guided layer and Pr~ as queries vs T_emb as keys/values in the context-guided layer (Eqs. 2 and 6)
- Alignment score is temperature-scaled by a learnable tau and trained with cross-entropy against ground-truth labels
- Optimizer settings reproduce the article's training regime: AdamW, lr 5e-5, weight decay 0.01, cosine scheduler, batch size 8 with gradient accumulation every 2 steps

## Limitations

- Future work aims to shorten the reasoning chains in EVL-MCoT while maintaining performance
- The standard CLIP model struggles with inputs exceeding 20 tokens, which is why Long-CLIP is required for the 150-word CoT sequences
- Repository README indicates only core training code is available; model definitions, configs, and pretrained weights are marked as coming soon

## Evidence

- [methods] context-guided decoders treat visual prototypes as queries and textual features as key-value pairs: [evidence span withheld — anchor status: not_found]
- [methods] Long-CLIP token context length: "we use Long-CLIP [26], which extends the context length to 248 tokens, enabling more effective reasoning over longer texts."
- [results] reported benchmark performance: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [readme] README status: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
- [methods] CLIP token limitation: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
