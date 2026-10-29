# domain-rag-citation-evaluation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **decompose_queries** — split each complex domain question into bounded retrieval subqueries  →  `subquery-decomposition-prompting`, `agricultural-query-refinement-llm`, `six-stage-retrieval-augmented-answer-synthesis`
2. **retrieve_evidence** — retrieve local and web evidence for every decomposed subquery  →  `parallel-multi-source-retrieval-orchestration`, `agricultural-query-decomposition-into-subqueries`, `web-retrieval-integration-with-vector-search`, `faiss-vector-search-retrieval`
3. **enhance_evidence** — inject selected domain-agent keywords into both retrieved evidence pools  →  `domain-agent-keyword-enhancement`, `web-search-llm-filtering`
4. **synthesize_answers** — generate evidence-grounded answers before citation insertion  →  `temperature-controlled-answer-synthesis`, `llm-response-quality-annotation-aggregation`, `llm-multi-agent-orchestration`
5. **insert_citations** — attach each supported answer sentence to a retrieved source chunk  →  `agricultural-answer-citation-insertion`, `sentence-embedding-similarity-scoring`, `citation-source-attribution`, `post-hoc-answer-grounding`
6. **score_grounded_answers** — aggregate human answer and citation judgments for the fixed query set  →  `citation-quality-scoring`, `composite-score-compilation-across-model-conditions`, `per-query-composite-score-analysis`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
