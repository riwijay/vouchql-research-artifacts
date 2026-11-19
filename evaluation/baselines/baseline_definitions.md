# Baseline Definitions

## 1. Purpose

This document defines the three experimental baselines used to evaluate
VouchQL:

1. LLM-Direct
2. VouchQL (No-Repair)
3. VouchQL (Full)

The purpose of these baselines is to compare direct LLM-based SQL generation
with the VouchQL approach and to isolate the contribution of its deterministic
repair harness.

## 2. LLM-Direct

LLM-Direct is the direct natural-language-to-SQL baseline.

In this baseline, the LLM generates SQL directly from the natural-language
question. The generated SQL is evaluated without the VouchQL deterministic
repair and validation pipeline.

This baseline represents the conventional approach in which SQL generation
is delegated directly to the language model.

The paper reports the following results across 100 evaluation questions:

| Metric | Result |
|---|---:|
| Structurally valid SQL statements | 73 / 100 |
| Structural errors | 27 / 100 |
| Healthy-question accuracy | 35.4% |
| Thin-question accuracy | 40.0% |
| Vacuous-question accuracy | 76.0% |
| Wrong-but-silent rate | 100% |

The evaluation identifies hallucinated tables and columns as examples of
structural failures in this baseline.

**Reproducibility note:** The paper does not provide a complete specification
of the LLM-Direct prompt, model configuration, decoding parameters, or
generation settings. These details must be supplied separately if available.

## 3. VouchQL (No-Repair)

VouchQL (No-Repair) is the VouchQL baseline without the deterministic repair
passes.

It provides a comparison point for examining the contribution of the repair
harness while retaining the VouchQL approach.

The paper illustrates this baseline with a question containing the vendor
name "Ciscoo". The unrepaired plan retains the misspelled value and executes
against it, returning zero rows.

The reported results across 100 evaluation questions are:

| Metric | Result |
|---|---:|
| Structurally valid SQL statements | 86 / 100 |
| Healthy-question accuracy | 36.9% |
| Thin-question accuracy | 60.0% |
| Vacuous-question accuracy | 56.0% |
| Wrong-but-silent rate | 0.0% |

The reported results show that removing deterministic repair reduces
structural validity and execution accuracy relative to the full VouchQL
configuration.

**Reproducibility note:** The paper identifies this baseline as VouchQL
without repair, but does not enumerate every disabled component or provide
a complete implementation-level configuration. Those details should be
documented if the experimental records are available.

## 4. VouchQL (Full)

VouchQL (Full) is the complete system evaluated in the paper.

It uses a structured QueryPlan as an intermediate representation. The
deterministic pipeline validates and repairs the plan before the compiler
generates SQL.

The system also supports human adjudication. Users can review a validated
plan and its generated SQL, then:

- Confirm the plan, storing it in the plan-level cache.
- Reject the plan, discarding it and invalidating the cache entry.
- Adjust the plan, modifying its attributes before it re-enters the
  compilation pipeline.

The full configuration includes the deterministic repair passes evaluated
in the paper.

The reported results across 100 evaluation questions are:

| Metric | Result |
|---|---:|
| Structurally valid SQL statements | 99 / 100 |
| Honest errors | 1 / 100 |
| Healthy-question accuracy | 41.5% |
| Thin-question accuracy | 70.0% |
| Vacuous-question accuracy | 52.0% |
| Wrong-but-silent rate | 2.4% |

The paper reports 41 incorrect answers among the 75 healthy and thin
questions. Of these, one was silent, while 40 surfaced a plan for
adjudication.

## 5. Baseline Comparison

| Metric | LLM-Direct | VouchQL (No-Repair) | VouchQL (Full) |
|---|---:|---:|---:|
| Valid SQL statements | 73 | 86 | 99 |
| Healthy accuracy | 35.4% | 36.9% | 41.5% |
| Thin accuracy | 40.0% | 60.0% | 70.0% |
| Vacuous accuracy | 76.0% | 56.0% | 52.0% |
| Wrong-but-silent rate | 100% | 0.0% | 2.4% |

The results indicate that VouchQL (Full) achieves the highest structural
validity and accuracy on healthy and thin questions among the evaluated
baselines.

The lower vacuous-question accuracy reflects a trade-off: the system favors
ontology-grounded behavior and explicit refusal over generating
valid-looking SQL for concepts that are empty or not adequately represented.

## 6. Interpretation and Limitations

The three baselines serve different experimental purposes:

- **LLM-Direct** provides the direct SQL-generation comparison.
- **VouchQL (No-Repair)** helps assess the contribution of deterministic
  repair.
- **VouchQL (Full)** evaluates the complete VouchQL design.

The reported metrics should be interpreted within the scope of the
100-question evaluation set and its healthy, thin, and vacuous partitions.

The paper does not provide all low-level configuration details needed to
reconstruct every baseline independently. Missing model, prompt, and
execution configuration details should be added from the original
experimental records rather than inferred.

