# Verify Before You Commit

Companion repository for *Verify Before You Commit: An Anomaly-Conditioned Diagnostic Benchmark for Structured-File Agents*. The benchmark studies whether an agent checks a structured input before relying on it, locates an injected integrity defect, and handles the evidence in its final answer.

## Dataset and evaluation scripts

The benchmark data and evaluation scripts are hosted on [Hugging Face](https://huggingface.co/datasets/anonymous-structured-agent/structured-file-audit-benchmark):

| Split | Source | Tasks | Files |
| --- | --- | ---: | --- |
| SC-Flat | DaBench | 256 | CSV |
| SC-Book | SpreadsheetBench | 120 | XLSX |
| SC-Pro | GDPval | 25 | CSV/XLSX professional-task pilot |

The release stores task files and evaluator-only manifests under `datasets/`, with split-specific scoring code under `eval_scripts/`. Each manifest records the injected defect and its localization anchors.

## Download

Install `huggingface_hub` and download the repository as files:

```bash
python -m pip install huggingface_hub
python - <<'PY'
from huggingface_hub import snapshot_download

snapshot_download(
    repo_id="anonymous-structured-agent/structured-file-audit-benchmark",
    repo_type="dataset",
    local_dir="benchmark_data",
)
PY
```

This file download preserves each task directory, including CSV files with task-specific columns.

## Read a task

After downloading, select a directory under `benchmark_data/datasets/<split>/data/`:

- **SC-Flat:** each task directory contains a perturbed CSV, its `.original.csv` counterpart, and `manifest.json`. The manifest includes the source task identifier; the original instruction comes from DaBench.
- **SC-Book:** each directory under `datasets/sc_book/data/tasks/` contains `prompt.txt`, a perturbed workbook, its `.original.xlsx` counterpart, and `manifest.json`.
- **SC-Pro:** each directory under `datasets/sc_pro/data/tasks/` contains `prompt.txt`, `task_metadata.json`, `manifest.json`, and task files.

Give the agent the task instruction and perturbed file. Use `manifest.json` and the original file for evaluator-side scoring. The scripts in `benchmark_data/eval_scripts/` are organized by split. Further reproduction materials will be added to this repository.

## Implementation and evaluation settings

The paper reports three model stacks (Opus 4.6, GLM-5, and Qwen3.5:27B) under a common Claude Code agent interface. Each is evaluated with the original task instruction (vanilla), a generic audit reminder (audit-simple), and an explicit coverage/plausibility/reconciliation checklist (audit-first). The complete prompt text and metric definitions are in the paper.

The released `eval_scripts/` use Python's standard library for trace parsing, manifest matching, and metric computation. Agent execution uses the Claude Agent SDK. The spreadsheet runner uses `pandas.read_excel` to prepare a workbook preview and selects its model through `ANTHROPIC_MODEL` when set.

The file-only reference validator uses fixed rules: a negative value in a non-negative column, a z-score above 3, an upper IQR fence of `Q3 + 1.5 * IQR`, or a modified MAD z-score above 3.5 for range defects; duplicate-row detection for duplication; a temporal gap above 2.5 times the median interval, a sequential-index jump above twice the typical step, or a missing referenced file for missing-block defects; and a summary discrepancy above 1% for reconciliation. The exact manifest anchors and perturbation parameters are stored per task.

## Provenance

The splits derive from DaBench, SpreadsheetBench, and GDPval. Their redistribution terms differ; see the [dataset card](https://huggingface.co/datasets/anonymous-structured-agent/structured-file-audit-benchmark) for source attribution and licensing details.
