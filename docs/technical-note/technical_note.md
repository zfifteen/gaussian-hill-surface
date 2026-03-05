# PhaseWall at the Gaussian Curvature Boundary: An Exploratory Technical Note

## Abstract
This note evaluates whether a phase-aware damping rule at the Gaussian curvature boundary ($r = \sigma$) is associated with better outcomes under noisy conditions. The evidence base now includes both a historical report-derived table and a new rerun pipeline with stricter statistical controls.

This revision addresses prior methodological concerns by using sign-robust effect metrics, two-sided paired tests, and multiple-comparison correction in the rerun analysis. Under the high-rigor rerun, `phasewall_tuned` is near-neutral in aggregate (2/36 cells with `q < 0.05`), while `lr_adapt_proxy` and `phasewall_plus_lr_tuned` show the strongest aggregate signals (34/36 and 33/36 cells with `q < 0.05`). A dedicated eval-only additive ablation (`lr_adapt_proxy` vs `phasewall_plus_lr_tuned` with fixed `s=0.1`) found 0/36 cells with pairwise BH-FDR `q < 0.05`, supporting closure of the standalone PhaseWall question under this protocol and pivoting primary interest to `lr_adapt_proxy`.

DOI status for this standalone repository release line:
- Version DOI (`v0.2.0`): `10.5281/zenodo.18856931`
- Concept DOI (standalone repository): `10.5281/zenodo.18856930`
- Legacy superseded DOI (monorepo): `10.5281/zenodo.18847306`

## 1. Research Question and Scope
### 1.1 Research Question
Can a phase-aware damping rule at the Gaussian curvature transition ($r = \sigma$) improve robustness and sample efficiency relative to a vanilla CMA-ES baseline under noisy benchmark conditions?

### 1.2 Scope Boundaries
In scope:
- Geometric boundary claim for the Gaussian graph surface.
- Rerun-derived benchmark claims under repository-defined protocol and artifacts.
- Historical report-derived results retained as context.

Out of scope:
- Universal claims across optimizers, tasks, or noise regimes.
- Production-readiness claims.
- Claiming exact reproduction of external LR-Adapt implementations (proxy caveat applies).

## 2. Method Definition
### 2.1 Radius Definitions
Isotropic normalized radius:

$$r = \lVert x - \mu \rVert / \sigma$$

Anisotropic extension (Mahalanobis radius):

$$r = \sqrt{(x - \mu)^T \Sigma^{-1} (x - \mu)}$$

### 2.2 Methods Compared in Rerun Pipeline
- `vanilla_cma`: pycma baseline.
- `lr_adapt_proxy`: explicit repository-local sigma adaptation proxy.
- `pop4x`: pycma with 4x population size.
- `phasewall_tuned`: PhaseWall with tuned strength `s`.
- `phasewall_plus_lr_tuned`: PhaseWall + LR proxy with tuned strength `s`.

For the high-rigor run documented here, tuned strengths selected by disjoint tune/eval protocol were:
- `phasewall_tuned`: `s = 0.4`
- `phasewall_plus_lr_tuned`: `s = 0.1`

## 3. Experimental Protocol (Rerun-Derived)
Run identity:
- Run ID: `20260305T002116Z-6ae43213`
- Scope: `high_rigor_rerun_pipeline`
- Config: `experiments/config/high_rigor.yaml`

Matrix:
- Functions: Sphere, Rosenbrock, Rastrigin, Ellipsoid (`cond=1e6`)
- Dimensions: 10, 20, 40
- Noise levels: `sigma_noise in {0.0, 0.1, 0.2}`

Execution protocol:
- Initial mean: `[3, 3, ..., 3]`
- Initial sigma: `2.0`
- Evaluation budget: `1,000` function evaluations per run
- Tuning seeds: `40`
- Evaluation seeds: `100`
- Tune/eval seeds are disjoint.

Tune-then-evaluate policy:
- Candidate strengths: `s in {0.1, 0.2, 0.4, 0.6, 0.8}`
- Tuning task subset: all functions, dimensions `{10,20}`, noise `{0.1}`
- Selection rule: minimize global median paired delta `(method - vanilla)`; tie-break by win-rate, then smaller `s`.

Primary statistics:
- Primary effect metric: `median_delta_vs_vanilla = median(method_final_best - vanilla_final_best)` (negative is better).
- Directional robustness: paired win/loss rates vs vanilla.
- Significance: two-sided Wilcoxon signed-rank test.
- Multiplicity: Benjamini-Hochberg FDR correction (`q` values) across method-cell tests.

Interpretation note:
- Ratio fields are treated as non-primary descriptors only.
- This avoids sign-inversion ambiguity when objective values can cross zero (e.g., noisy Sphere values can be negative due to additive noise).

## 4. Results (High-Rigor Rerun)
Execution integrity:
- Total run jobs: `21,520`
- Status: `21,520` ok, `0` failed
- Eval-phase jobs: `18,000`

Method-level aggregate summary (`36` cells per non-vanilla method):

| Method | median_of_cell_median_delta | mean_win_rate | cells_q_lt_0.05 | best_q_value |
|---|---:|---:|---:|---:|
| lr_adapt_proxy | -18.872089 | 0.505000 | 34 | 1.919845e-17 |
| phasewall_plus_lr_tuned | -18.836873 | 0.507778 | 33 | 1.919845e-17 |
| phasewall_tuned | -0.078369 | 0.506111 | 2 | 5.745819e-04 |
| pop4x | 58.695449 | 0.079722 | 36 | 1.919845e-17 |

Interpretation:
- `phasewall_tuned` is near-neutral-to-slightly-better in aggregate (`median_of_cell_median_delta = -0.078369`) and mixed across cells (21/36 cells with negative median delta).
- `lr_adapt_proxy` is the dominant observed driver in this matrix (`median_of_cell_median_delta = -18.872089`, 34/36 cells with `q < 0.05`).
- `phasewall_plus_lr_tuned` performs similarly to `lr_adapt_proxy` in aggregate under this protocol (33/36 cells with `q < 0.05`).
- `pop4x` degrades strongly relative to vanilla in this fixed-budget setting.

### 4.1 Additive Ablation (PW+LR vs LR)
To close the open decomposition question, we ran an eval-only additive ablation using fixed LR-proxy parameters and fixed `s = 0.1`:
- Run ID: `20260305T014110Z-321d79b1`
- Methods compared pairwise: `lr_adapt_proxy` (A) vs `phasewall_plus_lr_tuned` (B)
- Matrix and seeds: same 36-cell matrix, same 100 evaluation seeds, no tuning stage.

Pairwise result summary (`B - A`):
- Cells where B is better (negative median delta): `17/36`
- Cells where A is better (positive median delta): `19/36`
- Cells with uncorrected `p < 0.05`: `1/36`
- Cells with BH-FDR `q < 0.05`: `0/36`
- Median of cell medians (`B - A`): `0.002016829`

This ablation supports a practical closure statement for this note: adding PhaseWall on top of the LR proxy does not yield an isolable corrected-significant gain under the current protocol.

## 5. Reproducibility and Integrity Checks
Primary command used for this run:

```bash
bash scripts/run_high_rigor_pipeline.sh --workers 8
```

This command produces run-scoped artifacts under:
`artifacts/runs/high-rigor/<run_id>/`

Key machine-auditable artifacts for this run:
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/runs_long.csv`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/cell_stats.csv`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/method_aggregate.csv`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/pairwise_pwlr_vs_lr.csv`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/pairwise_pwlr_vs_lr.json`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/findings_ablation.md`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/manifest.json`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/analysis_manifest.json`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/findings.json`
- `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/findings.md`

Artifact verification command:

```bash
python3 scripts/verify_rerun_artifacts.py \
  --results-dir artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results \
  --figdir artifacts/runs/high-rigor/20260305T002116Z-6ae43213/figures \
  --config experiments/config/high_rigor.yaml \
  --mode full \
  --require-pairwise
```

Eval-only additive ablation command:

```bash
bash scripts/run_phasewall_ablation_pipeline.sh --workers 8
```

Ablation verification command:

```bash
python3 scripts/verify_rerun_artifacts.py \
  --results-dir artifacts/runs/phasewall-ablation/20260305T014110Z-321d79b1/results \
  --figdir artifacts/runs/phasewall-ablation/20260305T014110Z-321d79b1/figures \
  --config experiments/config/ablation_pwlr_vs_lr.yaml \
  --mode eval_only \
  --require-pairwise
```

## 6. Limitations
- `lr_adapt_proxy` is a transparent proxy, not an exact reproduction of external LR-Adapt implementations.
- Results are from one run configuration and should not be interpreted as universal across all possible settings.
- Tune/eval separation is enforced by disjoint seeds and task subset, but broader external validation is still needed.
- The tuning subset overlaps evaluation function families and one evaluation noise level (`sigma_noise = 0.1`), so some task-level leakage risk remains despite seed disjointness.
- Additive ablation in this note fixes PhaseWall at `s=0.1`; broader ablations over alternative coupling designs may still be explored separately.
- The geometric argument motivates the damping rule but does not by itself prove causal optimality on arbitrary objective families.

## 7. Conclusion
The curvature-sign claim at $r = \sigma$ remains mathematically exact (Appendix A). Empirically, under the high-rigor rerun protocol plus dedicated eval-only additive ablation, standalone PhaseWall evidence is weak and no corrected-significant incremental benefit is observed when adding PhaseWall to the LR-proxy variant (`0/36` pairwise cells with `q < 0.05`). The strongest reproducible signal in this repository now centers on `lr_adapt_proxy`. This note therefore closes the current standalone PhaseWall question under this protocol and treats LR-proxy characterization as the primary follow-on investigation direction. These are still scoped, non-universal claims.

## Appendix A: Curvature Sign Derivation for the Gaussian Hill
Let

$$z(x, y) = \exp\left(-(x^2 + y^2) / (2 \sigma^2)\right)$$

and $r^2 = x^2 + y^2$.

For a graph surface $z = f(x, y)$, Gaussian curvature is:

$$K = (f_{xx} f_{yy} - f_{xy}^2) / (1 + f_x^2 + f_y^2)^2$$

For this $z(x, y)$:

$$z_x = -(x / \sigma^2) z$$

$$z_y = -(y / \sigma^2) z$$

$$z_{xx} = ((x^2 - \sigma^2) / \sigma^4) z$$

$$z_{yy} = ((y^2 - \sigma^2) / \sigma^4) z$$

$$z_{xy} = (xy / \sigma^4) z$$

Compute the numerator:

$$z_{xx} z_{yy} - z_{xy}^2$$

$$= (z^2 / \sigma^8)\left[(x^2 - \sigma^2)(y^2 - \sigma^2) - x^2 y^2\right]$$

$$= (z^2 / \sigma^8)\left[\sigma^4 - \sigma^2 (x^2 + y^2)\right]$$

$$= (z^2 / \sigma^6)(\sigma^2 - r^2)$$

The denominator $(1 + z_x^2 + z_y^2)^2$ is strictly positive, so the sign of $K$ is the sign of $(\sigma^2 - r^2)$:
- $r < \sigma$ -> $K > 0$
- $r = \sigma$ -> $K = 0$
- $r > \sigma$ -> $K < 0$

Therefore, the curvature sign change occurs exactly at $r = \sigma$.

## References
1. `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/findings.md`
2. `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/runs_long.csv`
3. `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/cell_stats.csv`
4. `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/method_aggregate.csv`
5. `artifacts/runs/high-rigor/20260305T002116Z-6ae43213/results/pairwise_pwlr_vs_lr.csv`
6. `artifacts/runs/phasewall-ablation/20260305T014110Z-321d79b1/results/pairwise_pwlr_vs_lr.csv`
7. `docs/analysis/lr_adapt_proxy_mechanism.md`
8. `docs/analysis/lr_adapt_proxy_breakdown.md`
9. `experiments/config/high_rigor.yaml`
