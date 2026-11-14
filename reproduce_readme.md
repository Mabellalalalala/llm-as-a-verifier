# FTEC5660 Reproducibility Experiment

This repository contains a small reproducibility study of **LLM-as-a-Verifier** on Terminal-Bench 2.1.

The study compares:

- **Baseline:** Best-of-3 candidate selection
- **Modification:** Best-of-5 candidate selection
- **Verifier:** DeepSeek `deepseek-flash`
- **Dataset:** Existing Terminal-Bench 2.1 trajectories

The experiment is intentionally limited to one complete task, `cancel-async-tasks`, to keep API cost and execution time within the assignment budget.

## Installation

Use the system Python installation; a virtual environment is not required.

```bash
cd /Users/mabel/individual2/llm-as-a-verifier
python3 -m pip install --upgrade pip
python3 -m pip install -e .
```

The project depends on:

- `openai`
- `google-genai`
- `tqdm`

If PyPI is unavailable, pass the mirror explicitly:

```bash
python3 -m pip install -e . \
  --index-url https://pypi.tuna.tsinghua.edu.cn/simple \
  --trusted-host pypi.tuna.tsinghua.edu.cn
```

## API configuration

Create a local `.env` file. It is ignored by Git and must never be committed:

```dotenv
DEEPSEEK_API_KEY=your_key_here
```

The verifier uses:

```text
Base URL: https://api.deepseek.com
Model: deepseek-flash
```

Do not put real keys in source files, notebooks, logs, screenshots, or commits.

## Dataset and selected task

The repository includes the Terminal-Bench 2.1 trajectories under:

```text
data/terminal_bench_2.1_trajs/mini-swe-agent_deepseek-v4-flash/
```

The selected task list is stored in:

```text
experiments/selected_tasks.json
```

Current pilot task:

```json
["cancel-async-tasks"]
```

## Running the experiments

Both runs use the same task list, prompts, criteria, model, seed, repeated evaluations, and pivot count. Only the candidate count changes.

### Baseline: Best-of-3

```bash
python3 scripts/run_tb2.1_bo3.py \
  --tasks-file experiments/selected_tasks.json \
  --seed 0 \
  --n-evaluations 2 \
  --max-workers 1 \
  2>&1 | tee results/baseline/run.log
```

### Modification: Best-of-5

```bash
python3 scripts/run_tb2.1_bo5.py \
  --tasks-file experiments/selected_tasks.json \
  --seed 0 \
  --n-evaluations 2 \
  --max-workers 1 \
  2>&1 | tee results/modified/run.log
```

### Seed sensitivity check

The same commands can be repeated with `--seed 1`. The seed-1 runs reuse cached verifier scores when available.

## Smoke test

Before running the benchmark, test API connectivity and candidate selection with a small two-candidate example. The expected result selects the function returning `2` and saves a cache file at `cache/smoke_test.json`.

## Results

Generated artifacts include:

- `results/repository_terminal_bench_2.1_mswea_deepseek_bo3.txt`
- `results/repository_terminal_bench_2.1_mswea_deepseek_bo5.txt`
- `results/repository_comparison.csv`
- `results/repository_comparison.json`
- `results/repository_comparison.png`
- `results/repository_flow_diagram.png`

The pilot result was 100% verifier success for both Bo-3 and Bo-5. Bo-5 used more verifier calls and tokens without improving selection on the selected task. Because this is a one-task pilot, the result should not be interpreted as a full Terminal-Bench benchmark result.

## Changes made for the study

Changes are intentionally small and reviewable:

1. Added optional `--tasks-file` filtering to the Terminal-Bench Bo-3 and Bo-5 runners so both experiments can use exactly the same task subset.
2. Added `experiments/selected_tasks.json` to record the selected task IDs.
3. Added result and report artifacts under `results/`.

The verifier scoring algorithm, prompts, dataset files, and tournament implementation were preserved.

If committing these changes, use small, separate commits such as:

```text
Add task-list filtering for reproducibility subsets
Record selected Terminal-Bench tasks
Add experiment result artifacts and report figures
```

Review each commit before pushing. No commit should contain `.env`, API keys, or other credentials.

## Reproducibility notes

- Python: 3.12
- `seed`: 0 for the main runs; 1 for sensitivity analysis
- `n_evaluations`: 2
- `pivots`: 1
- `max_workers`: 1
- Criteria: `specification`, `output_match`, `error_signals`

Network failures should be recorded as failed runs. A result is valid only when the output reports nonzero verifier calls and token usage.
