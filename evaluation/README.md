# VouchQL Evaluation Dataset

## 1. Overview

This directory contains the evaluation question set and supporting documentation for the VouchQL research project.

VouchQL is a deterministic NL2SQL harness that constrains the authority of a Large Language Model (LLM) through a structured intermediate QueryPlan. A deterministic Python-based harness validates, repairs, and compiles the plan into SQL. The evaluation examines SQL execution behavior, structural validity, silent failures, and the contribution of individual deterministic repair passes.

The released evaluation dataset provides the natural-language questions, expected SQL queries, expected intent labels, and supporting evaluation metadata used to describe the experimental protocol.

The dataset is released as a research artifact. It does not include the proprietary VouchQL implementation or the original enterprise database.

## 2. Directory Contents

```text
evaluation/
├── README.md
├── evaluation_protocol.md
├── questions/
│   └── evaluation_questions.yaml
├── baselines/
│   └── baseline_definitions.md
├── metrics/
│   └── metric_definitions.md
├── scripts/
└── results/
```

The principal dataset is:

`questions/evaluation_questions.yaml`

It contains five top-level sections:

| Section | Description |
|---|---|
| `questions` | Healthy evaluation questions |
| `thin_questions` | Questions associated with thin tables or subsets |
| `vacuous_questions` | Questions associated with empty tables or non-discriminating concepts |
| `cache_pairs` | Canonical question variants used to examine normalization and plan-level caching |
| `ablation_subset` | Question IDs associated with individual deterministic repair passes |

## 3. Dataset Composition

The evaluation question set contains 100 questions distributed across three categories.

| Category | Number of questions | Evaluation purpose |
|---|---:|---|
| Healthy | 65 | Execution accuracy and structural validity |
| Thin | 10 | Execution accuracy reported separately |
| Vacuous | 25 | Structural validity |
| **Total** | **100** | |

The categories retain the classification used in the experimental dataset.

### 3.1 Healthy Questions

Healthy questions target tables containing at least 100 rows and are intended to provide meaningful result variation.

They are used for execution-accuracy evaluation.

### 3.2 Thin Questions

Thin questions target tables containing between 1 and 99 rows or relevant subsets.

They are evaluated separately from healthy questions to distinguish behavior on smaller datasets.

### 3.3 Vacuous Questions

Vacuous questions target empty tables or non-discriminating concepts.

They are included in structural-validity evaluation. They are not included in the execution-accuracy denominator.

The original dataset's classifications are preserved in this release. They have not been reclassified as part of the anonymization process.

## 4. Question Record Structure

Question records contain the following fields:

| Field | Description |
|---|---|
| `id` | Stable question identifier |
| `q` | Natural-language question |
| `expected_intent` | Expected query intent, such as `count`, `list`, or `group_count` |
| `golden_sql` | Reference SQL query used for comparison |
| `order_sensitive` | Indicates whether row ordering matters during result comparison |
| `source` | Indicates the question's source classification |
| `notes` | Supporting information, assumptions, or review annotations |

The question IDs and evaluation categories are retained to preserve traceability across the released evaluation artifacts.

### 4.1 Golden SQL

The `golden_sql` field provides the reference SQL query associated with a natural-language question.

Execution accuracy is determined by comparing the result produced for a question against the result produced by its reference SQL.

The comparison uses exact row-set matching, with ordering considered when `order_sensitive` is enabled.

The reference SQL is provided as evaluation data. It does not imply that the released repository includes the database or execution environment required to execute it.

## 5. Cache-Pair Dataset

The `cache_pairs` section contains 12 canonical question-variation pairs.

These pairs are intended to examine question normalization and plan-level cache behavior.

Each pair includes:

- A reference question identifier (`pair_of`).
- A natural-language question variant.
- An expected intent.
- An `expect_same_plan` indicator.
- A source classification.
- Supporting notes where applicable.

Cache pairs are not additional execution-accuracy questions and are not included in the 100-question dataset total.

The cache-pair cases distinguish between variations expected to resolve to the same QueryPlan and variations expected to produce a different plan.

For example, the dataset includes a typo-variation case intended to test whether a misspelled vendor reference can resolve to the same canonical plan.

## 6. Ablation Subset

The `ablation_subset` section identifies questions associated with individual deterministic repair passes.

The released YAML contains:

- 26 entries.
- 24 unique question IDs.

The distinction is intentional in this release: the original ordering and repeated entries are preserved rather than silently removing duplicates.

Two question IDs occur more than once:

| Question ID | Occurrences |
|---|---:|
| `q040` | 2 |
| `q041` | 2 |

These repeated entries appear under different repair-pass annotations in the YAML.

The YAML's original descriptive comment indicating 25 IDs has been updated to reflect the actual entry count and unique-ID count.

The research paper reports an ablation experiment involving 24 questions. The 24 unique IDs in this released subset are consistent with that number. However, the existence of 24 unique IDs alone does not establish that this exact version of the YAML was the precise input used to produce every reported ablation result.

The ablation list is preserved as research metadata; it should not be interpreted as 26 independent experimental questions.

## 7. Evaluation Metrics

The evaluation protocol distinguishes structural validity from execution accuracy.

### 7.1 Structural Validity

A result is structurally valid when:

1. The evaluation result reports `result.ok == True`.
2. The generated SQL executes successfully.

Structural validity is evaluated across all three question categories.

### 7.2 Execution Accuracy

Execution accuracy is based on exact result matching against the corresponding `golden_sql` result.

The healthy and thin categories are evaluated separately.

Vacuous questions are excluded from the execution-accuracy denominator.

### 7.3 Wrong-but-Silent Failures

A wrong-but-silent failure occurs when:

- The generated result does not match the reference result; and
- The incorrect plan is not surfaced for human adjudication.

This metric is computed over healthy and thin questions only.

### 7.4 Wrong-but-Visible Failures

A wrong-but-visible failure occurs when the generated result is incorrect but the plan is surfaced for human adjudication.

This distinction is important because a system that exposes uncertainty or incorrect plans for review behaves differently from one that silently returns incorrect results.

For the complete metric definitions and denominators, see:

`metrics/metric_definitions.md`

For the experimental procedure, see:

`evaluation_protocol.md`

## 8. Anonymization and Data Protection

The released question set has undergone value-level pseudonymization of selected enterprise-specific values.

The anonymization process preserves the technical and semantic structure of the evaluation dataset.

### 8.1 Preserved Information

The following information is retained:

- Table names and column names.
- SQL joins and relationships.
- SQL functions, filters, and aggregation logic.
- Question IDs and question categories.
- Expected intent labels.
- Ordering requirements.
- Evaluation metadata and annotations.
- Aggregate counts and evaluation classifications.

These elements are necessary to understand the evaluation structure and SQL semantics.

### 8.2 Pseudonymized Information

Business-specific values are replaced with consistent synthetic identifiers.

Examples include:

- Vendor names and references.
- Site and location names.
- Device serial numbers.
- Selected enterprise-specific values appearing in natural-language questions, SQL literals, and notes.

Pseudonyms are applied consistently across the question text, reference SQL, cache-pair records, and supporting notes.

The original enterprise identifiers and their replacement mapping are not included in this public repository.

### 8.3 Semantic Preservation

Pseudonymization is intended to preserve the original query structure and expected evaluation intent.

It does not independently establish that every replacement value exists in the original database or that every query produces an identical result after substitution.

Researchers should therefore distinguish between:

- Preservation of SQL structure and evaluation metadata; and
- Reproducibility of the original database execution results.

The original enterprise database is not part of this release.

## 9. Reproducibility and Limitations

The released dataset supports inspection of the evaluation questions, reference SQL, category definitions, cache-pair cases, and ablation metadata.

Full experimental reproduction requires additional components that are not included in this public repository.

These include:

- The VouchQL execution and repair harness.
- The database used for the original experiments.
- The required ontology and semantic artifacts.
- The model and inference configuration.
- The evaluation environment and dependencies.
- The exact experimental scripts and configuration used to produce the reported results.

Consequently, the public dataset should be treated as a supplementary research artifact, not as a standalone executable reproduction package.

The absence of the proprietary implementation and original enterprise database is deliberate.

## 10. Research Use

Researchers may inspect, analyze, and reference the released evaluation data subject to the repository's applicable license or copyright terms.

When using this dataset, please cite the associated VouchQL research paper.

For questions about the evaluation methodology, dataset structure, or research artifacts, please refer to the documentation in this repository.
