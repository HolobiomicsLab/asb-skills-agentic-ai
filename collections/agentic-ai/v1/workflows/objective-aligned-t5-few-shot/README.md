# objective-aligned-t5-few-shot-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **prepare_adaptation_data** — turn commonsense text into objective-labeled unsupervised adaptation samples  →  `omcs-sentence-splitting-adaptation-data-preparation`, `seq2seq-sentence-splitter-training`, `sentence-splitter-training-on-knowledge-graph-tuples`
2. **format_few_shot_examples** — format knowledge-graph tuples as aligned few-shot input-target examples  →  `atomic2020-knowledge-completion-data-formatting`, `task-category-classification`, `head-tail-pair-classifier-training`
3. **adapt_checkpoint** — continue pre-training a T5 checkpoint with a task-aligned denoising objective  →  `denoising-objective-adaptation-training`, `unsupervised-adaptation-data-preparation`, `objective-aligned-adaptation-pretraining`, `denoising-vs-lm-objective-alignment`
4. **finetune_kb_completion** — fine-tune the adapted checkpoint on masked few-shot knowledge completion  →  `t5-encoder-decoder-fine-tuning`, `pretraining-objective-template-alignment`, `few-shot-qa-fine-tuning`, `fine-tuning-template-alignment`
5. **evaluate_few_shot_strategy** — compare the aligned checkpoint with a supervised few-shot baseline  →  `few-shot-evaluation-of-language-models`, `rouge-bertscore-generation-evaluation`, `commonsense-knowledge-completion-evaluation`, `rouge-generation-metric-computation`, `generation-plausibility-metric-scoring`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
