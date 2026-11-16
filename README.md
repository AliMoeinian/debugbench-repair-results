<p align="center">
  <a href="https://ui.ac.ir/"><img src="https://upload.wikimedia.org/wikipedia/commons/4/42/University_of_Isfahan_Logo.png" alt="University of Isfahan logo" width="84" height="83"></a>
  <br>
  <strong>University of Isfahan</strong>
</p>

<h1 align="center">DebugBench C++ Repair Results</h1>

<p align="center">
  Supplementary evaluation data for<br>
  <strong>Reflexion and Self-Debugging as Single-Agent Design Patterns for C++ Program Repair on DebugBench</strong>
</p>

<p align="center">
  <strong>4 models</strong> · <strong>3 repair configurations</strong> · <strong>1,438 C++ instances</strong> · <strong>17,256 evaluation records</strong>
</p>

<p align="center">
  <a href="#study-overview"><img src="https://img.shields.io/badge/Study%20overview-145DA0?style=for-the-badge" alt="Study overview"></a>
  <a href="#results-at-a-glance"><img src="https://img.shields.io/badge/Results%20at%20a%20glance-00796B?style=for-the-badge" alt="Results at a glance"></a>
  <a href="#files"><img src="https://img.shields.io/badge/Files-6B46A1?style=for-the-badge" alt="Files"></a>
  <a href="#how-to-read-the-data"><img src="https://img.shields.io/badge/How%20to%20read%20the%20data-9E4A13?style=for-the-badge" alt="How to read the data"></a>
  <a href="#authors-and-citation"><img src="https://img.shields.io/badge/Authors%20and%20citation-45556F?style=for-the-badge" alt="Authors and citation"></a>
</p>

---

## Study overview

Does giving a local small language model a chance to revise its repair help? We compare the original DebugBench direct-prompting baseline with compiler-guided adaptations of **Reflexion** and **Self-Debugging**. The same 1,438 C++ instances were used for every model and configuration. Local syntax checks guided revision in the iterative configurations; **LeetCode Online Judge (LOJ)** assessed the final retained candidate independently, without feeding its verdict back into generation.

## Results at a glance

Acceptance is the share of all evaluation records marked `Accepted` by LOJ. The denominator is **1,438** per model and configuration, and **5,752** per pooled configuration.

| Model | Direct prompting | Reflexion | Self-Debugging |
| :--- | ---: | ---: | ---: |
| Qwen2.5-Coder-7B-Instruct | 60.92% | 68.08% | **68.78%** |
| Llama-3.1-8B-Instruct | **38.60%** | 33.80% | 11.20% |
| Granite-8B-Code-Instruct-4K | 23.71% | 24.20% | **25.94%** |
| Mistral-7B-Instruct-v0.3 | 4.38% | **26.36%** | 25.80% |
| **Pooled across all four models** | **1,835 / 5,752 (31.90%)** | **2,192 / 5,752 (38.11%)** | **1,894 / 5,752 (32.93%)** |

Reflexion had the highest pooled acceptance. Qwen with Self-Debugging had the highest acceptance among individual model–configuration combinations, while Mistral with Reflexion improved the most over its own baseline. Both patterns reduced acceptance for Llama. The iterative runs also used more recorded tokens, so acceptance alone does not tell the whole story.

## Files

| File | What it contains |
| :--- | :--- |
| [`results.csv`](results.csv) | 17,256 rows, one per `(model, configuration, instance_id)`. |
| [`configuration_summary.csv`](configuration_summary.csv) | Counts, acceptance rates, token totals, and recorded generation time for each model and configuration. |
| [`paired_summary.csv`](paired_summary.csv) | Paired acceptance counts for each iterative configuration versus the same model’s baseline. |
| [`SHA256SUMS.txt`](SHA256SUMS.txt) | SHA-256 checksums for the three CSVs and this README. |

On systems with `sha256sum`, verify the files from the repository root:

```bash
sha256sum -c SHA256SUMS.txt
```

## How to read the data

- **Overall acceptance:** Count `outcome == "Accepted"` and divide by all rows in the group. The dataset contains **16,641 final LOJ verdicts**, **611 generation failures**, and **4 technically unresolved evaluations**. Generation failures and unresolved evaluations remain in the overall denominator.
- **Paired comparisons:** Join within each model on `instance_id`. Exclude a pair if either side is technically unresolved; otherwise, count a generation failure as non-accepted. Each pooled comparison has **5,750 eligible pairs**. Net gain is pattern-only acceptances minus baseline-only acceptances: **+357** for Reflexion and **+60** for Self-Debugging. The latter is **+59** when comparing the full, unpaired acceptance totals.
- **Resource measurements:** Recorded generation latency excludes LOJ execution. Baseline and iterative runs differed in hardware and timing scope, so these values should not be read as a controlled head-to-head latency measurement.
- **Scope:** The CSVs let readers recompute the released outcome and resource summaries. They are not a rerun package. The repository does not include benchmark code, generated programs, per-request prompts, raw logs, submission IDs, or hidden tests. Some diagnostics in the manuscript were derived from retained artifacts outside this release.

<details>
<summary><strong>Column guide for results.csv</strong></summary>

The CSV is UTF-8 with a header row. Blank means unavailable or not applicable; zero is a recorded value.

| Column | Meaning |
| :--- | :--- |
| `instance_id` | Study instance identifier. Use this to pair results instead of the problem slug, which can recur across instances. |
| `problem_slug`, `language`, `difficulty`, `bug_category`, `bug_subtype` | Benchmark problem and defect metadata. |
| `model`, `configuration` | Model label and repair configuration. |
| `outcome` | End-to-end outcome. Possible values are Accepted, Wrong Answer, Compile Error, Runtime Error, Time Limit Exceeded, Memory Limit Exceeded, Generation failure, and Technical unresolved. |
| `loj_verdict` | Final judge verdict. Blank for generation failure and technically unresolved evaluation. |
| `local_compile_passed` | Recorded local syntax-check flag, available for the iterative configurations. In Self-Debugging it can refer to the preceding candidate if the last candidate was not rechecked. |
| `code_generation_attempts` | Recorded iterative candidate-generation attempts (1–3), excluding reflection-only calls and technical retries. Blank for the baseline, which retained one candidate. An attempt need not produce a nonempty candidate. |
| `generation_retry_count` | Recorded technical generation retries on the evaluation record. This is distinct from the candidate-generation attempt count. |
| `prompt_tokens`, `completion_tokens`, `total_tokens` | Recorded model-request usage, including reflection usage where applicable. Failed requests without returned usage are excluded. |
| `generation_latency_seconds` | Recorded generation latency in seconds, excluding LOJ time. |
| `passed_test_count`, `total_test_count` | Recorded LOJ test counts where available. |
| `loj_runtime_ms`, `loj_memory_mb` | Reported judge execution time in milliseconds and memory in MB. |
| `extraction_method`, `extracted_code_length`, `malformed_tag_order` | Source extraction label, recorded extracted code length, and a flag for malformed tag order. |
| `generation_error_recorded`, `context_limit_error_recorded` | Flags for a recorded generation error and a context-related message, respectively. An error flag can occur even when a program was retained. |

</details>

## Authors and citation

[Moein Aldin AliHosseini](https://orcid.org/0009-0008-9723-8088) (equal contribution), [Ali Moeinian](https://orcid.org/0009-0002-1232-3379) (equal contribution), [Afsaneh Fatemi](https://orcid.org/0000-0003-1184-5917) (corresponding author), and [Mohammad Ali Nematbakhs](https://orcid.org/0000-0002-4374-9228).

Department of Computer Software Engineering, University of Isfahan, Isfahan, Iran.

If you use these evaluation records, please cite the accompanying study and reference this repository. The upstream benchmark is [DebugBench](https://huggingface.co/datasets/Rtian/DebugBench), introduced by Tian et al. in [*Findings of ACL 2024*](https://doi.org/10.18653/v1/2024.findings-acl.247). This repository contains the study’s evaluation results on the C++ subset of that benchmark.

<sub>University of Isfahan logo: <a href="https://commons.wikimedia.org/wiki/File:University_of_Isfahan_Logo.png">Wikimedia Commons</a>.</sub>
