# VouchQL: A Deterministic NL2SQL Harness

### Supplementary Research Artifacts — ICDE 2027

**Paper title:**

*VouchQL: A Deterministic NL2SQL Harness with Bounded LLM Authority, Plan-Level Caching, and Human-in-the-Loop Semantic Adjudication*

**Authors:**
- Riky — PT Mastersystem Infotama Tbk, Jakarta, Indonesia
- Robby Saleh — BINUS University, Jakarta, Indonesia

---

## 1. Overview

VouchQL is a deterministic Natural Language to SQL (NL2SQL) harness designed to improve the structural validity, reliability, and auditability of LLM-based database interfaces.

Rather than allowing a Large Language Model (LLM) to generate SQL directly, VouchQL constrains the LLM to produce a structured intermediate query plan.

A deterministic compiler subsequently validates, repairs, and translates the plan into executable SQL.

The architecture incorporates three principal mechanisms:

1. **Bounded LLM Authority:** Restricting LLM output to a structured intermediate representation rather than executable SQL.
2. **Deterministic Repair and Compilation:** Applying rule-based structural repairs and validation before SQL execution.
3. **Human-in-the-Loop Semantic Adjudication:** Allowing users to confirm, reject, or adjust query plans before they become reusable knowledge.

Confirmed query plans are persisted through plan-level caching, enabling deterministic replay without repeated LLM inference.

## 2. Research Contributions

This research investigates the following contributions:

- A structured intermediate representation (QueryPlan) separating semantic planning from SQL generation.
- A deterministic repair harness addressing observed LLM planning errors.
- Structural validation before SQL execution.
- Human-in-the-loop semantic adjudication.
- Plan-level caching with ontology and semantic artifact fingerprinting.
- An empirical evaluation of structural validity, silent failures, execution accuracy, and cache replay performance.

## 3. Experimental Results

The accompanying research paper reports the following results:

| Evaluation | Reported result |
|---|---:|
| Evaluation questions | 100 |
| Structural SQL validity — LLM-Direct | 73% |
| Structural SQL validity — VouchQL | 99% |
| Wrong-but-silent failure rate — LLM-Direct | 100% |
| Wrong-but-silent failure rate — VouchQL | 2.4% |
| Warm-cache benchmark questions | 65 |
| Total warm-cache replay time | 0.148 seconds |
| Average warm-cache replay latency | 2.29 milliseconds/query |
| Reported latency improvement | >13,800× |

These figures are reported results from the accompanying manuscript. Their underlying experimental records and calculation procedures will be provided in the evaluation artifacts where available.

## 4. Repository Structure

```text
vouchql-research-artifacts/
│
├── README.md
├── CITATION.cff
│
├── environment/
│
├── evaluation/
│   ├── questions/
│   ├── baselines/
│   ├── metrics/
│   ├── scripts/
│   └── results/
│
├── examples/
│
├── reference/
│
└── figures/
```

## 5. Research Artifacts

The repository is intended to provide supporting materials for reviewing the research methodology and experimental findings.

Planned artifact categories include:

| Category | Description |
|---|---|
| Evaluation protocol | Experimental methodology and evaluation procedure |
| Evaluation questions | Sanitized research evaluation inputs |
| Baseline definitions | Descriptions of experimental comparison baselines |
| Metric definitions | Definitions and calculation procedures for reported metrics |
| Experimental results | Anonymized or sanitized evaluation results |
| QueryPlan examples | Illustrative intermediate query representations |
| Architecture documentation | Conceptual description of the VouchQL architecture |
| Experimental configuration | Hardware, model, and inference configuration |

Artifact availability and reproduction limitations will be documented individually.

## 6. Implementation and Intellectual Property

This repository contains research supplementary materials, not the complete VouchQL production implementation.

The proprietary implementation, internal enterprise database, production semantic artifacts, deployment packages, and operational components are intentionally excluded.

The published materials are intended to support academic inspection of the research methodology, architecture, and experimental evidence without disclosing the complete underlying software system.

## 7. Reproducibility

The evaluation materials are provided to support independent examination of the reported experimental methodology and results.

Reproduction requirements, dependencies, execution instructions, and any limitations will be documented alongside the corresponding artifacts.

Where original enterprise data or proprietary implementation components cannot be distributed, this limitation will be explicitly stated.

## 8. Citation

If you use or reference this research, please cite:

```text
Riky and Robby Saleh,
"VouchQL: A Deterministic NL2SQL Harness with Bounded LLM
Authority, Plan-Level Caching, and Human-in-the-Loop
Semantic Adjudication,"
Submitted to IEEE ICDE 2027.
```

## 9. Contact

For questions concerning the research artifacts, please contact the corresponding authors through their institutional contact channels.

---

**Research supplementary materials for VouchQL — ICDE 2027.**
