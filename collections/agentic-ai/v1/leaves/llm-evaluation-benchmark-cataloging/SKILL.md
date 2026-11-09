---
name: llm-evaluation-benchmark-cataloging
description: Use when a survey section on Autonomous Evaluation (Dynamic Benchmarking
  §6.2 / Interactive Environment Evaluation §6.3) contains tabular benchmark/environment
  listings that must be converted into structured records keyed by name, update mode,
  reward type, task domain, and metrics.
when_to_use_negative:
- The source is prose discussion of evaluation bottlenecks (§7.5) rather than the
  actual catalog tables — there are no rows to parse into records
- The tables list training datasets (e.g. BookCorpus, FineWeb-Edu, The Stack v2 from
  the intro/methods data-acquisition sections) instead of evaluation benchmarks/environments
- Only one table is needed in isolation when the research question requires the process-based
  vs outcome-based classification that links Table 11 rows to the Autonomous Evaluation
  component
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Python
  role: Parses the line-wrapped citation strings in Table 10/11 rows and writes the
    structured records into separate CSV files with explicit header lines
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:2da414270373cc2ed011d6f3a8a4dfa33a61bebfd466dd39890cce1ff9a17788
  source_papers:
  - doi: 10.48550/arxiv.2603.25681
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# llm-evaluation-benchmark-cataloging

## Summary

Extracts and structures the paper's Autonomous Evaluation catalog tables — Table 10 (dynamic benchmarks such as RealTime QA, LiveCodeBench, OKBench) and Table 11 (interactive environments such as WebArena, AppWorld, AndroidWorld, Lean-Gym, SafeArena) — into validated CSV records so the taxonomy's process-based vs outcome-based classification is machine-readable.

## When to use

Use when a survey section on Autonomous Evaluation (Dynamic Benchmarking §6.2 / Interactive Environment Evaluation §6.3) contains tabular benchmark/environment listings that must be converted into structured records keyed by name, update mode, reward type, task domain, and metrics.

## When NOT to use

- The source is prose discussion of evaluation bottlenecks (§7.5) rather than the actual catalog tables — there are no rows to parse into records
- The tables list training datasets (e.g. BookCorpus, FineWeb-Edu, The Stack v2 from the intro/methods data-acquisition sections) instead of evaluation benchmarks/environments
- Only one table is needed in isolation when the research question requires the process-based vs outcome-based classification that links Table 11 rows to the Autonomous Evaluation component

## Inputs

- Table 10 (dynamic evaluation benchmarks) rows: benchmark name, update mode (live/static), data origin
- Table 11 (interactive environments) rows: environment name, outcome/process reward type, task domain, evaluation metrics
- Article section Autonomous Evaluation (§6) including §6.2 Dynamic Benchmarking and §6.3 Interactive Environment Evaluation

## Outputs

- CSV file of Table 10 records with explicit header line and one row per dynamic benchmark (e.g. RealTime QA, LiveCodeBench, OKBench)
- CSV file of Table 11 records with explicit header line and one row per interactive environment (e.g. WebArena, AppWorld, AndroidWorld, Lean-Gym, SafeArena)

## How to apply

First locate the dynamic-benchmarking table (Table 10) and record its caption and column headers covering benchmark name, update mode (live/static), and data origin, then parse each row (e.g. RealTime QA, LiveCodeBench, OKBench) into structured records, handling line-wrapped citation strings so authors like '(Polu et al., 2022)' stay attached to the correct environment. Next locate the interactive-environments table (Table 11), capture headers for environment name, outcome/process reward type, task domain, and evaluation metrics, and parse rows such as WebArena, AppWorld, AndroidWorld, Lean-Gym, and SafeArena preserving every row and attribute. Write the two record sets into separate CSV files with explicit header lines and one row per benchmark/environment. Preserve the taxonomy's classification: Lean-Gym and SafeArena are process-based interactive environments while AppWorld and AndroidWorld are outcome-based, all under the Autonomous Evaluation component. Finally, validate row counts and field completeness against the source tables before finalizing both files.

## Related tools

- **Python** (Parses the line-wrapped citation strings in Table 10/11 rows and writes the structured records into separate CSV files with explicit header lines)

## Examples

```
python -c "import pandas as pd; t11=pd.read_csv('table11_interactive_envs.csv'); assert t11[['environment','reward_type','domain','metrics']].notna().all().all()"
```

## Evaluation signals

- Row count of each output CSV equals the row count of the corresponding source table (Table 10 and Table 11)
- Every CSV row has non-empty values for all header fields (benchmark name, update mode, data origin; or environment name, reward type, task domain, metrics)
- Line-wrapped citations are rejoined correctly — each environment keeps its full citation (e.g. Lean-Gym (Polu et al., 2022), SafeArena (Tur et al., 2025), AppWorld (Trivedi et al., 2024), AndroidWorld (Rawles et al., 2024))
- Classification check: Lean-Gym and SafeArena are tagged process-based; AppWorld and AndroidWorld are tagged outcome-based
- Both CSV files contain explicit header lines as the first line before any data records

## Limitations

- Line-wrapped citation strings in the source tables can split author-year references across lines and must be handled explicitly during parsing
- The catalog reflects the paper's snapshot of §6 (dynamic benchmarking and interactive environments); the README notes the paper list is continuously updated, so newer benchmarks may be missing
- Field completeness validation depends on the source tables themselves being fully populated; missing cells propagate to the CSV records

## Evidence

- [results] taxonomy citation for Autonomous Evaluation environments: [evidence span withheld — anchor status: not_found]
- [methods] table parsing workflow: [evidence span withheld — anchor status: not_found]
- [readme] README section structure: "- [Autonomous Evaluation (§6)](#autonomous-evaluation-6)
  - [Dynamic Benchmarking (§6.2)](#dynamic-benchmarking-62)
  - [Interactive Environment Evaluation"
- [intro] evaluation component definition: [evidence span withheld — anchor status: not_found]
