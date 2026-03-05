# Rerun Smoke Report

This report summarizes the smoke-stage artifact generation for the pycma rerun pipeline.

## Scope
- Full matrix exercised across all methods.
- Smoke settings only (not final inferential campaign):
  - 10 eval seeds per cell
  - 200 evals per run

## Command

```bash
bash scripts/run_smoke_pipeline.sh
```

## Outputs
- `artifacts/results/rerun-sample/runs_long.csv`
- `artifacts/results/rerun-sample/tuning_summary.csv`
- `artifacts/results/rerun-sample/selected_params.json`
- `artifacts/results/rerun-sample/cell_stats.csv`
- `artifacts/results/rerun-sample/method_aggregate.csv`
- `artifacts/results/rerun-sample/findings.json`
- `artifacts/results/rerun-sample/findings.md`
- `artifacts/figures/rerun-sample/method_median_delta_bar.png`
- `artifacts/figures/rerun-sample/method_win_rate_bar.png`
- `artifacts/figures/rerun-sample/method_q_lt_005_count_bar.png`

## Pipeline Summary
- Selected strengths:
  - `phasewall_tuned`: `s = 0.6`
  - `phasewall_plus_lr_tuned`: `s = 0.2`
- Total runs recorded: `2680`
  - eval: `1800`
  - tune_baseline: `80`
  - tune_candidate: `800`
- Run status: all `ok` in smoke execution.
- Cell-level comparator rows in `cell_stats.csv`: `144` (`36` cells x `4` non-vanilla methods).

## Smoke-Stage Statistical Snapshot
- Primary tests: two-sided Wilcoxon paired by seed.
- Multiplicity: BH-FDR (`q` values).
- Rows with `q < 0.05` in smoke run: `30`.

Method aggregate highlights from `method_aggregate.csv`:
- `lr_adapt_proxy`: median cell delta `-44.345596`, cells with `q < 0.05`: `2`.
- `phasewall_plus_lr_tuned`: median cell delta `-14.797942`, cells with `q < 0.05`: `1`.
- `phasewall_tuned`: median cell delta `0.370344`, cells with `q < 0.05`: `0`.
- `pop4x`: median cell delta `1403.262301`, cells with `q < 0.05`: `27`.

## Notes
- Smoke artifacts validate schema, pipeline wiring, and statistical post-processing.
- Findings are run-scoped and tied to the smoke run manifest via `run_id`.
- Smoke-stage findings are not used to replace DOI-facing canonical claims.
- Full high-rigor reruns (100 eval seeds/cell, 1000 eval budget) are pending.
