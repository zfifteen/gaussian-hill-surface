# Rerun Protocol (pycma, High-Rigor Track)

## Goal
Build a reproducible rerun pipeline that addresses peer-review concerns around metric interpretation, inferential robustness, comparator completeness, and hyperparameter governance.

## Methods
- `vanilla_cma`
- `lr_adapt_proxy`
- `pop4x`
- `phasewall_tuned`
- `phasewall_plus_lr_tuned`

### LR-Adapt proxy caveat
`lr_adapt_proxy` is an explicit repository-local proxy, not an exact reproduction of Nomura et al. or the historical bundled report implementation. All outputs retain `proxy` naming to avoid overclaiming.

Proxy sigma update (per generation):
1. `signal = max(previous_best - current_best, 0)`
2. `noise = 1.4826 * MAD(fitness_generation) + eps`
3. `snr = signal / noise`
4. `ema_snr = alpha * snr + (1-alpha) * ema_snr_prev`
5. If `ema_snr < down_threshold`, apply `sigma *= sigma_down_factor`.
6. Else if `ema_snr > up_threshold`, apply `sigma *= sigma_up_factor`.
7. Clamp sigma to `[sigma_min_ratio * sigma0, sigma_max_ratio * sigma0]`.

## PhaseWall definition
For candidate displacement `d = y - mean`, compute Mahalanobis radius in current CMA state:

`r = sqrt(((d/sigma)^T C^{-1} (d/sigma)))`

Let `r0 = sqrt(dimension - 2/3)`. For `r > r0`:

`a = clip(1 - s * (1 - r0/r), 0, 1)`

Evaluate at `mean + a * d`, but call `tell` with original undamped candidate and damped-point fitness.

## Matrix
- Functions: `sphere`, `rosenbrock`, `rastrigin`, `ellipsoid_cond1e6`
- Dimensions: `10, 20, 40`
- Noise: `0.0, 0.1, 0.2`

## Tune-then-evaluate split (strict)
- Tuning tasks: all functions, dimensions `{10,20}`, noise `{0.1}`
- Tuning seeds and evaluation seeds are disjoint by config and validated at runtime.
- Candidate strengths: `s in {0.1, 0.2, 0.4, 0.6, 0.8}`

Selection rule per tuned method:
1. Minimize global median paired delta `(method - vanilla)` over tuning tasks/seeds.
2. Tie-breaker: higher win-rate vs vanilla.
3. Tie-breaker: smaller `s`.

## Statistical outputs
Primary columns:
- `median_delta_vs_vanilla` (negative is better)
- `win_rate_vs_vanilla`
- `loss_rate_vs_vanilla`
- `wilcoxon_p_two_sided`
- `bh_fdr_q_value`

A ratio column may be emitted as non-primary descriptor only.

## CLI
- Run pipeline: `python3 -m experiments.run --config <yaml> --outdir <dir>`
- Analyze runs: `python3 -m experiments.analyze --runs <runs_long.csv> --outdir <dir> --manifest-json <manifest.json>`
- Generate findings: `python3 -m experiments.findings --results-dir <dir> --figdir <figdir>`
- Smoke end-to-end: `python3 -m experiments.smoke --config experiments/config/smoke.yaml`
- High-rigor wrapper: `bash scripts/run_high_rigor_pipeline.sh [--workers N]`

## Run Identity Contract
Each pipeline invocation is assigned a stable run identifier:

- Format: `YYYYMMDDTHHMMSSZ-<config_hash8>`
- Stored in `manifest.json` as `run_id`
- Propagated to `analysis_manifest.json` as `run_id`
- Referenced by `findings.json`/`findings.md`

Additional manifest metadata:
- `run_scope` (from `experiment_name`)
- `run_root` (artifact root path for that invocation)
- `git_commit` (short commit SHA, or `unknown`)

## Findings Artifact Contract
Each run results directory emits:

- `findings.json` (machine-readable)
- `findings.md` (human-readable)

These findings are run-scoped and must link back to:
- `manifest.json`
- `analysis_manifest.json`
- run-generated CSV/figure artifacts
