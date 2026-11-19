# Experimental Configuration

## 1. Purpose

This document records the hardware, software, inference, database, and
execution configuration reported for the VouchQL experiments.

The purpose is to make the experimental environment understandable and
to identify configuration details that are necessary for independent
reproduction.

Only configuration details explicitly reported in the research paper are
presented as established facts. Missing information is marked as
"Not reported" and should be completed only after verification against
the original experimental records.

## 2. Hardware Environment

The evaluation stack was deployed on a standard developer laptop.

| Component | Reported configuration |
|---|---|
| Device | Lenovo Yoga Slim 7 Pro X |
| CPU | AMD Ryzen 7 6800HS |
| GPU | NVIDIA GeForce RTX 3050 |
| GPU memory | 4 GB |
| System memory | Not reported |
| Storage | Not reported |

The reported deployment demonstrates that the VouchQL stack can operate
on developer-laptop hardware without requiring a server-class system.

## 3. Language Model

| Parameter | Reported configuration |
|---|---|
| Model | LFM2.5-8B-A1B |
| Quantization | Q4_K_M |
| Model format | GGUF |
| Inference engine | llama.cpp |
| Server executable | llama-server |
| Reasoning mode | Disabled |

The model is served locally. The paper reports an offline deployment
configuration.

The exact model file checksum and model distribution source are
not reported in the paper.

## 4. Inference Server Configuration

The paper reports the following inference-server command:

```bash
GGML_CUDA_DISABLE_GRAPHS=1 llama-server \
  -m LFM2.5-8B-A1B-Q4_K_M.gguf \
  --jinja -fa on --fit on --load-mode none \
  -t 8 -tb 8 -c 24576 -ctk q8_0 -ctv q8_0 -np 1 \
  --reasoning off --host localhost --port 8083
### 4.1 Parameter Reference

| Parameter | Reported value | Description |
|---|---|---|
| `GGML_CUDA_DISABLE_GRAPHS` | `1` | Disables CUDA graphs |
| `-m` | `LFM2.5-8B-A1B-Q4_K_M.gguf` | Model file |
| `--jinja` | Enabled | Enables Jinja chat templates |
| `-fa` | `on` | Enables flash attention |
| `--fit` | `on` | Enables automatic fit behavior |
| `--load-mode` | `none` | Reported model loading mode |
| `-t` | `8` | Inference threads |
| `-tb` | `8` | Batch-processing threads |
| `-c` | `24576` | Context size |
| `-ctk` | `q8_0` | K cache quantization |
| `-ctv` | `q8_0` | V cache quantization |
| `-np` | `1` | Parallel slots |
| `--reasoning` | `off` | Disables reasoning output |
| `--host` | `localhost` | Listening host |
| `--port` | `8083` | Listening port |

The exact llama.cpp version and build configuration used for the
reported evaluation are not reported in the paper.

## 5. Database Environment

| Parameter | Reported configuration |
|---|---|
| Database engine | SQLite |
| SQLite version | 3.45.1 |
| Database file | Not reported |
| Number of tables | Not reported |
| Number of rows | Not reported |
| Schema size | Not reported |

The evaluation uses an enterprise-grounded database domain.

The paper reports 100 held-out questions partitioned into:

- 65 healthy questions, targeting tables with at least 100 rows.
- 10 thin questions, targeting small tables or subsets.
- 25 vacuous questions, probing empty tables or non-discriminating
  concepts.

The evaluation is limited to a single enterprise database domain.

## 6. Plan Cache Configuration

The plan cache is implemented as a flat JSON file named
`plan_cache.json`.

| Parameter | Reported configuration |
|---|---|
| Cache representation | JSON |
| Cache file | `plan_cache.json` |
| Cache key | Normalized question |
| Cache value | Validated QueryPlan |
| Persistence | Written atomically on confirmation |
| Invalidation fingerprint | Ontology, semantic, and glossary files |

A confirmed query plan is reused for identical normalized questions.
A cache hit bypasses the LLM components and routes the stored plan
directly to the compiler.

Changes to the ontology, semantic, or glossary artifacts invalidate
the cache through the artifact fingerprint mechanism.

## 7. Deployment Architecture

The paper describes the deployment as an offline Docker-based stack
containing:

- FastAPI web server.
- Python compiler pipeline.
- SQLite database.
- Locally served LLM through llama.cpp.

The deployment is intended for enterprise data-center environments
that may be air-gapped or lack GPU provisioning.

Exact Docker image digests, container resource limits, operating-system
versions, and Python package versions are not reported in the paper.

## 8. Benchmark Execution and Cache Conditions

The paper reports a replay benchmark over 65 schema-grounded questions.

| Measurement | Reported result |
|---|---:|
| Number of replayed questions | 65 |
| Average cold-run latency | 31,637 ms/query |
| Total warm-cache replay time | 0.148 seconds |
| Average warm-cache latency | 2.29 ms/query |
| SQL replay consistency | 100% byte-identical |

The cold run includes LLM planning and validation. A warm cache hit
bypasses LLM inference.

The paper also reports a total cold-run time of 2,619 seconds and a
total warm-cache replay time of 0.148 seconds.

**Reproducibility note:** The reported average cold-run latency and
reported total cold-run time should be checked against the original
benchmark logs. This document preserves both reported figures rather
than attempting to reconcile them.

The paper does not report the complete benchmark script, individual
per-query latency records, warm-up procedure, or detailed cache-reset
procedure.

## 9. Software Versions and Experimental Environment

The following environment details were collected from the original
machine used for the reported experiments.

### 9.1 Hardware and Operating System

| Item | Experimental configuration |
|---|---|
| System | Lenovo Yoga Slim 7 Pro X |
| Architecture | x86_64 |
| CPU | AMD Ryzen 7 6800HS Creator Edition |
| Logical CPUs | 16 |
| System memory | 27 GiB reported by the operating system |
| GPU | NVIDIA GeForce RTX 3050 |
| GPU memory | 4 GB |
| Operating system | Ubuntu 24.04.5 LTS |
| Linux kernel | 7.0.0-43-generic |

### 9.2 Software Environment

| Item | Experimental configuration |
|---|---|
| Python | 3.12.3 |
| SQLite | 3.45.1 |
| Docker | 29.9.3 (build 5457199) |
| NVIDIA driver | 595.99.02 |
| CUDA version reported by NVIDIA driver | 13.2 |
| CUDA compiler toolkit | 12.0 (V12.0.140) |
| llama-server | 0.1.0-dev |
| llama-server build | Build 0, commit unknown |

The CUDA version reported by NVIDIA-SMI represents the CUDA
compatibility level reported by the installed driver. The CUDA
compiler toolkit version is reported separately by `nvcc`.

The llama-server version output identifies the executable as a
development build but does not provide a usable source commit
identifier. Consequently, the exact llama.cpp source revision
cannot be established from the available version output.

### 9.3 Configuration Details Not Available

The following details were not established by the available
experimental records and should not be inferred.

| Item | Status |
|---|---|
| llama.cpp source commit / exact revision | Not reported |
| SQLite library build options | Not reported |
| Python dependency versions | Not reported |
| Database schema fingerprint | Not reported |
| Model SHA-256 checksum | Not reported |
| Evaluation script revision | Not reported |
| Random seed, if applicable | Not reported |

These fields should be completed only when supported by retained
configuration files, build records, execution logs, or other
experimental evidence.

## 10. Reproducibility and Data Availability

The research artifact repository is intended to provide the
methodological and experimental information needed to understand
the evaluation without disclosing proprietary implementation or
enterprise data.

The following items may be provided separately when cleared for
publication:

- Sanitized evaluation questions.
- Evaluation protocol and metric definitions.
- Baseline definitions.
- Aggregated experimental results.
- Sanitized QueryPlan examples.
- Evaluation scripts that do not expose proprietary implementation.
- Environment and dependency manifests.

The original enterprise database, production semantic artifacts,
proprietary source code, credentials, and deployment bundles are
not included in this repository.

Any synthetic or sanitized dataset must be clearly identified and
must not be represented as the original enterprise evaluation data.
