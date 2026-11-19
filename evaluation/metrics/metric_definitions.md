# VouchQL Metric Definitions

## 1. Purpose

This document defines the evaluation metrics used in the VouchQL research study.

The metrics distinguish structural SQL validity, semantic correctness, silent failure behavior, and cache replay performance.

They are intended to make the reported evaluation results interpretable and to clarify the boundaries of each measurement.

## 2. Structural Validity

### 2.1 Definition

Structural validity measures whether a generated SQL statement satisfies the structural requirements necessary for execution.

A structurally valid statement is distinguished from a statement that fails because of issues such as hallucinated tables, nonexistent columns, invalid joins, or SQL syntax errors.

Structural validity does not independently establish that the query expresses the user's intended meaning.

### 2.2 Formula

**Structural Validity Rate**

$$
\frac{\text{Valid SQL Statements}}{\text{Total Questions}} \times 100
$$

**Structural Failure Rate**

$$
\frac{\text{Structural Failures}}{\text{Total Questions}} \times 100
$$

### 2.3 Reported Results

For the 100-question evaluation:

| Baseline | Valid SQL | Structural Failures | Validity Rate |
|---|---:|---:|---:|
| LLM-Direct | 73 | 27 | 73% |
| VouchQL (No-Repair) | 86 | 14 | 86% |
| VouchQL (Full) | 99 | 1 | 99% |

The reported structural failure rate decreases from 27% for LLM-Direct to 1% for VouchQL (Full).

## 3. Wrong-but-Silent Failure Rate

### 3.1 Definition

A wrong-but-silent failure occurs when the system returns an incorrect result without surfacing the query plan for user review.

This metric focuses on the visibility of incorrect results rather than simply counting all incorrect answers.

An incorrect result that is exposed through the query plan for human adjudication is distinguished from an incorrect result returned without such visibility.

### 3.2 Evaluation Population

The metric is evaluated on the 75 healthy and thin questions.

The 25 vacuous questions are excluded.

### 3.3 Formula

$$
\text{Wrong-but-Silent Rate} =
\frac{\text{Silent Failures}}{\text{Incorrect Answers}} \times 100
$$

### 3.4 Reported Results

| Baseline | Incorrect Answers | Silent Failures | Wrong-but-Silent Rate |
|---|---:|---:|---:|
| LLM-Direct | 25 | 25 | 100% |
| VouchQL (Full) | 41 | 1 | 2.4% |

For VouchQL (Full), 40 of the 41 incorrect answers surfaced a plan for adjudication, while one remained silent.

**Interpretation:** This metric measures the visibility of incorrect answers. It should not be interpreted as an overall semantic accuracy score.

## 4. Execution Accuracy

### 4.1 Definition

Execution accuracy measures the correctness of query results across the three evaluation categories:

- Healthy questions.
- Thin questions.
- Vacuous questions.

The evaluation reports separate accuracy values for each category.

### 4.2 Reported Results

| Baseline | Healthy | Thin | Vacuous |
|---|---:|---:|---:|
| LLM-Direct | 35.4% | 40.0% | 76.0% |
| VouchQL (No-Repair) | 36.9% | 60.0% | 56.0% |
| VouchQL (Full) | 41.5% | 70.0% | 52.0% |

### 4.3 Interpretation

The results show different behavior across the three categories.

VouchQL (Full) reports higher accuracy than LLM-Direct on healthy and thin questions.

On vacuous questions, LLM-Direct reports higher accuracy. The paper attributes this difference to LLM-Direct producing valid-looking SQL against empty tables, while VouchQL may refuse or return zero when the requested concept is absent or insufficiently represented in the ontology.

This demonstrates a trade-off between raw execution accuracy and conservative handling of unsupported or vacuous concepts.

## 5. Deterministic Repair Ablation

### 5.1 Definition

The ablation experiment examines the contribution of selected deterministic repair passes by disabling them individually.

The experiment uses a 24-question subset.

### 5.2 Metrics

Three outcomes are reported:

- **Correct:** Number of questions producing correct answers.
- **Wrong:** Number of questions producing incorrect answers.
- **Error:** Number of structural execution errors.

### 5.3 Reported Results

| Configuration | Correct | Wrong | Structural Errors |
|---|---:|---:|---:|
| Full System | 14 | 10 | 0 |
| Pass 1 Disabled: Column Relocation | 12 | 6 | 6 |
| Pass 11 Disabled: Profile Expansion | 12 | 9 | 3 |
| Pass 12 Disabled: Key-Value Anchors | 12 | 9 | 3 |

These results are reported for the selected 24-question subset and should not be generalized directly to the entire 100-question evaluation.

## 6. Plan-Level Cache Performance

### 6.1 Definition

Plan-level cache performance measures the latency of replaying previously confirmed natural-language questions using stored validated QueryPlan objects.

A cache hit bypasses the LLM planning stage and sends the stored plan directly to the deterministic compiler.

The cache key is the normalized natural-language question.

### 6.2 Reported Measurements

| Metric | Reported Value |
|---|---:|
| Warm-cache replay questions | 65 |
| Cold-run average latency | 31,637 ms/query |
| Warm-cache total latency | 0.148 seconds |
| Warm-cache average latency | 2.29 ms/query |
| Byte-identical SQL output | 100% |

### 6.3 Latency Reduction

The latency reduction factor can be calculated as:

$$
\text{Latency Reduction Factor} =
\frac{\text{Cold Average Latency}}{\text{Warm Average Latency}}
$$

Using the reported average values:

$$
\frac{31,637}{2.29} \approx 13,815.7
$$

This is consistent with the paper's reported reduction of greater than 13,800 times.

However, the paper also reports a cold-run total of 2,619 seconds for the 65-question benchmark. This total does not reconcile with the reported average of 31,637 ms/query.

The original benchmark records must be checked before publishing a definitive cold-run total or recalculated comparison.

### 6.4 SQL Output Consistency

The paper reports 100% byte-identical SQL output during warm-cache replay.

This measures consistency of the compiled SQL output across the replayed questions. It does not independently establish semantic correctness of the original query plan.

## 7. Metric Limitations

The reported metrics have several important boundaries:

1. Structural validity does not prove semantic correctness.
2. Execution accuracy depends on the evaluation dataset and its reference answers.
3. Wrong-but-silent rate measures incorrect answers that are not surfaced for adjudication, not all possible system failures.
4. The ablation results apply to a 24-question subset.
5. Cache latency measurements are specific to the reported experimental configuration.
6. Byte-identical SQL output establishes output consistency, not correctness of the underlying result.
7. Aggregate results alone do not provide enough information to independently reconstruct every per-question outcome.

## 8. Data and Reproducibility Requirements

For independent recalculation of these metrics, the following materials would be useful:

- Per-question baseline outcomes.
- Question category assignments.
- Reference answers or an appropriately sanitized correctness oracle.
- Structural validation outcomes.
- Semantic correctness judgments.
- Human adjudication outcomes.
- Cold-run and warm-cache timing records.
- Cache replay output comparisons.
- Ablation configuration and per-question outcomes.

Where these records cannot be released, the public artifact repository should state the limitation explicitly.

---

**Document status:** Research artifact draft.

**Source:** VouchQL research paper, Evaluation and Plan-Level Caching sections.
