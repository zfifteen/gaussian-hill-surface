# Run Findings

## Run Identity
- Run ID: `20260305T001143Z-2d89856b`
- Scope: `smoke_rerun_pipeline`
- Created (UTC): `2026-03-05T00:12:46.683459+00:00`
- Config: `experiments/config/smoke.yaml`
- Config Hash: `2d89856b9bdac6fe242a7a34adb16cf229daef0d405bb241d76eabe000aa990f`
- Manifest: `artifacts/results/rerun-sample/manifest.json`
- Analysis Manifest: `artifacts/results/rerun-sample/analysis_manifest.json`

## Execution Integrity
- Total runs: `2680`
- OK runs: `2680`
- Failed runs: `0`
- Status by phase:
  - `eval` / `ok`: `1800`
  - `tune_baseline` / `ok`: `80`
  - `tune_candidate` / `ok`: `800`

## Tuning Outcomes
- `phasewall_plus_lr_tuned` selected strength: `0.2`
- `phasewall_tuned` selected strength: `0.6`

## Statistical Findings
- Cell rows: `144`
- Methods in aggregate: `4`
- Rows with BH-FDR q < 0.05: `30`
- Method ranking (lower median delta is better):
  - `lr_adapt_proxy`: median delta `-44.34559581418432`, mean win-rate `0.5305555555555557`, q<0.05 cells `2`
  - `phasewall_plus_lr_tuned`: median delta `-14.797942336913636`, mean win-rate `0.5333333333333333`, q<0.05 cells `1`
  - `phasewall_tuned`: median delta `0.3703440277286017`, mean win-rate `0.4583333333333333`, q<0.05 cells `0`
  - `pop4x`: median delta `1403.2623011332582`, mean win-rate `0.1166666666666666`, q<0.05 cells `27`

## Caveats
- LR-Adapt comparator is a transparent proxy implementation, not exact Nomura reproduction.
- Findings are run-scoped; treat smoke runs as pipeline-validation evidence, not final inferential evidence.

## Artifact Links
- `analysis_manifest_json`: `artifacts/results/rerun-sample/analysis_manifest.json`
- `cell_stats_csv`: `artifacts/results/rerun-sample/cell_stats.csv`
- `figure_method_delta`: `artifacts/figures/rerun-sample/method_median_delta_bar.png`
- `figure_method_q_count`: `artifacts/figures/rerun-sample/method_q_lt_005_count_bar.png`
- `figure_method_win_rate`: `artifacts/figures/rerun-sample/method_win_rate_bar.png`
- `manifest_json`: `artifacts/results/rerun-sample/manifest.json`
- `method_aggregate_csv`: `artifacts/results/rerun-sample/method_aggregate.csv`
- `runs_long_csv`: `artifacts/results/rerun-sample/runs_long.csv`
- `selected_params_json`: `artifacts/results/rerun-sample/selected_params.json`
- `tuning_summary_csv`: `artifacts/results/rerun-sample/tuning_summary.csv`
