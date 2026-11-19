# VouchQL Evaluation Protocol

## 1. Purpose

This document describes the evaluation methodology used to assess VouchQL, a deterministic NL2SQL harness with bounded LLM authority, plan-level caching, and human-in-the-loop semantic adjudication.

The evaluation examines three principal properties:

1. Structural validity of generated SQL.
2. Wrong-but-silent failure behavior.
3. Execution accuracy and the role of human adjudication.

Additional experiments examine the contribution of selected deterministic repair passes and the performance characteristics of plan-level caching.

The evaluation follows the experimental configuration and results reported in the accompanying VouchQL research paper.

## 2. Evaluation Dataset

The evaluation uses 100 held-out, enterprise-grounded natural-language questions.

The questions are partitioned into three categories:

| Category | Number of Questions | Description |
|---|---:|---|
| Healthy | 65 | Questions targeting tables with at least 100 rows. |
| Thin | 10 | Questions targeting small tables or subsets. |
| Vacuous | 25 | Questions probing empty tables or non-discriminating concepts. |
| **Total** | **100** | |

This partition is intended to distinguish ordinary data retrieval from queries involving limited or absent underlying data.

The vacuous category is particularly relevant when examining the difference between producing executable SQL and producing an answer that is semantically appropriate for the underlying data.

## 3. Evaluation Baselines

Three system configurations are evaluated.

### 3.1 LLM-Direct

The LLM-Direct baseline generates SQL directly from the natural-language question.

It represents the conventional LLM-to-SQL approach without VouchQL's deterministic query-plan compilation boundary.

### 3.2 VouchQL (No-Repair)

This configuration uses VouchQL's structured query-planning and deterministic compilation architecture without the deterministic repair passes enabled.

It provides a comparison point for assessing the contribution of the repair mechanism.

### 3.3 VouchQL (Full)

This configuration uses the complete VouchQL pipeline, including structured query planning, deterministic validation, repair passes, SQL compilation, and human-in-the-loop semantic adjudication.

## 4. Evaluation Dimensions

### 4.1 Structural Validity

Structural validity measures whether the system produces SQL that passes the structural execution requirements.

The evaluation reports the number of valid SQL statements for each baseline.

The reported results are:

| Baseline | Valid SQL | Structural Failures |
|---|---:|---:|
| LLM-Direct | 73 | 27 |
| VouchQL (No-Repair) | 86 | 14 |
| VouchQL (Full) | 99 | 1 |

The structural failure rate is calculated as:

Structural Failure Rate = (Number of Structural Failures / Total Questions) × 100

For the 100-question evaluation, the reported structural failure rates are 27% for LLM-Direct and 1% for VouchQL (Full).

### 4.2 Wrong-but-Silent Failure Rate

The wrong-but-silent failure rate measures incorrect answers that are returned without surfacing the query plan for user review.

This metric is evaluated on the 75 healthy and thin questions. The 25 vacuous questions are excluded.

The paper reports:

| Baseline | Incorrect Answers | Silent Failures | Silent Rate |
|---|---:|---:|---:|
| LLM-Direct | 25 | 25 | 100% |
| VouchQL (Full) | 41 | 1 | 2.4% |

The metric is calculated as:

Wrong-but-Silent Rate = (Number of Silent Failures / Number of Incorrect Answers) × 100

For VouchQL, 40 of the 41 incorrect answers surfaced a query plan for adjudication. One incorrect answer was silent.

This metric distinguishes an incorrect result that is exposed for review from an incorrect result that is returned without such visibility.

### 4.3 Execution Accuracy

Execution accuracy is reported separately for healthy, thin, and vacuous questions.

The paper reports the following results:

| Baseline | Healthy Accuracy | Thin Accuracy | Vacuous Accuracy |
|---|---:|---:|---:|
| LLM-Direct | 35.4% | 40.0% | 76.0% |
| VouchQL (No-Repair) | 36.9% | 60.0% | 56.0% |
| VouchQL (Full) | 41.5% | 70.0% | 52.0% |

The results illustrate a trade-off between execution accuracy and conservative handling of concepts that are absent or insufficiently represented in the underlying ontology.

In particular, the lower vacuous-question accuracy reported for VouchQL reflects its tendency to refuse or return zero when the requested concept cannot be appropriately mapped, rather than producing valid-looking SQL against unsuitable data.

## 5. Deterministic Repair Ablation

A separate ablation experiment uses a subset of 24 questions.

The experiment examines the effect of disabling selected deterministic repair passes.

The reported results are:

| Configuration | Correct | Wrong | Structural Errors |
|---|---:|---:|---:|
| Full System | 14 | 10 | 0 |
| Pass 1 Disabled: Column Relocation | 12 | 6 | 6 |
| Pass 11 Disabled: Profile Expansion | 12 | 9 | 3 |
| Pass 12 Disabled: Key-Value Anchors | 12 | 9 | 3 |

The ablation examines whether selected repair passes contribute to structural validity and semantic correctness.

The reported examples include validator failures, incomplete column selection, and omitted filter constraints.

## 6. Plan-Level Cache Benchmark

The cache experiment evaluates replay latency across 65 schema-grounded questions.

The cache stores validated QueryPlan objects rather than generated SQL strings.

The cache key is the normalized natural-language question. A fingerprint of the ontology, semantic, and glossary artifacts is used to invalidate cached entries when those artifacts change.

The reported benchmark results are:

| Measurement | Reported Result |
|---|---:|
| Number of replay questions | 65 |
| Cold-run average latency | 31,637 ms per query |
| Warm-cache total replay time | 0.148 seconds |
| Warm-cache average latency | 2.29 ms per query |
| SQL output consistency | 100% byte-identical |

A warm cache hit bypasses the LLM planning stage and sends the stored validated plan directly to the compiler.

The experiment reports a latency reduction greater than 13,800 times.

The paper also reports a cold-run total of 2,619 seconds. The relationship between this total and the reported average cold-run latency should be clarified in the underlying benchmark records before publishing independently recalculated comparisons.

## 7. Human-in-the-Loop Adjudication

VouchQL provides three user interaction paths:

1. **Confirm:** The user confirms the validated query plan. The plan is stored in the cache for subsequent identical questions.
2. **Reject:** The user rejects the plan. The plan is discarded, and the cache entry is invalidated so that a subsequent attempt goes through the LLM again.
3. **Adjust Plan:** The user modifies plan attributes directly. The updated plan bypasses the LLM and re-enters the pipeline at the compiler stage.

These interactions establish the user as the semantic authority before a plan is persisted for reuse.

## 8. Reproducibility and Data Availability

The evaluation was conducted using an enterprise-grounded database environment.

The public research artifact repository is intended to provide supporting methodological documentation, metric definitions, sanitized examples, and non-sensitive experimental results.

The original enterprise database, production semantic artifacts, proprietary implementation, and deployment components are not included in the public artifact repository.

Consequently, the public materials do not provide unrestricted reproduction of the original enterprise evaluation.

Any additional released evaluation inputs or scripts must be clearly identified as sanitized, synthetic, or derived materials, as appropriate.

## 9. Experimental Details Requiring Further Documentation

The research paper reports the evaluation dataset sizes, baseline configurations, principal metrics, and aggregate results.

The following details should be documented from the original experimental records before claiming full independent reproducibility:

- Exact natural-language questions and their reference answers.
- The procedure used to determine correctness for each question.
- The exact LLM configuration and inference parameters used for each baseline.
- The precise execution and adjudication procedure.
- The hardware and software configuration used during evaluation.
- The exact procedure used to collect cold-run and warm-cache timings.
- The per-question evaluation records and calculation scripts.
- The exact membership and selection procedure for the 24-question ablation subset.

Missing information should be reported as unavailable rather than reconstructed from assumptions.

---

**Document status:** Research artifact draft.

**Source:** VouchQL research paper, Evaluation, Plan-Level Caching, and Human-in-the-Loop sections.
