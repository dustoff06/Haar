# Reference-Measure Geometry in Quantum Parameter Estimation

**Replication code for:**
> Fulton, C. P. & Fulton, L. V. (2026). *Reference-Measure Geometry in Quantum Parameter Estimation: When Coordinate Surrogates Produce Ridge-Side Convergence.* Submitted to *Physical Review A*.

---

## Overview

This repository contains the complete replication code for the computational study in the paper. The central finding is that combining intrinsically defined likelihoods (Born-rule) with coordinate-space regularization — while intending Haar isotropy — implicitly replaces the Haar reference measure with Lebesgue measure, specifying a *different statistical model*. This reference-measure mismatch modifies the gradient field and stationary-point structure of the optimization objective, producing ridge-side convergence in realistic gate-estimation pipelines.

The code implements two regularized MAP objectives for single-qubit gate estimation on SU(2):

```
L_E(v) = -log p(data | U(v)) + (λ/2)||v||²           # Euclidean / flat
L_G(v) = -log p(data | U(v)) + (λ/2)||v||² - log J(v) # Haar-consistent
```

where `J(v) = [sin(||v||/2) / (||v||/2)]²` is the exponential-map chart-volume (Haar Jacobian) factor.

---

## Repository Structure

```
.
├── researchnotefinal_02182026.ipynb   # Main replication notebook
├── su2_pra_table2_results_1700.csv    # Raw results (1,700 runs, 5 configs × 340 starts)
└── README.md
```

---

## Key Components

### SU(2) Utilities
- `axis_angle_to_su2(v)` — Exponential map from axis-angle coordinates to SU(2)
- `haar_jacobian(v)` — Chart-volume factor J(v) with numerically stable near-identity evaluation
- `grad_log_jacobian(v)` — Gradient of log J(v); near identity: ∇ log J(v) ≈ −v/6

### Objectives
- `loss_flat(v, params)` — Euclidean MAP objective L_E
- `loss_geom(v, params)` — Haar-consistent objective L_G
- Both with analytic gradients via central-difference NLL + closed-form Jacobian correction

### Experiment
- Multi-start L-BFGS-B optimization across 5 configurations spanning likelihood-dominated, balanced, prior-sensitive, and stress regimes
- 340 random initializations per configuration drawn from v₀ ~ N(0, 0.09 I₃)
- Box constraints vᵢ ∈ [−3, 3] to maintain chart validity
- 11,900 total converged runs analyzed across the full study

### Ridge-Side Diagnostics
For each flat-coordinate solution v*_E satisfying ||∇L_E(v*_E)|| < 10⁻⁵:

1. **Geometric gradient magnitude** `||∇L_G(v*_E)||` — measures violation of Haar stationarity
2. **Descent certificate** — verifies strict decrease L_G(v*_E − η∇L_G) < L_G(v*_E) for interior points
3. **Intrinsic loss gap** — L_G(v*_E) − L_G(v*_G) from independent geometric re-optimization
4. **Gate infidelity** — 1 − F(U_true, U*) under both objectives

### Output Tables
`make_extreme_tables()` produces:
- Top-10 interior extremes ranked by ||∇L_G(v*_E)||
- Top-10 boundary extremes (constraint-dominated control)
- Per-configuration summary with boundary incidence and gradient statistics

---

## Reproducing the Paper Results

### Requirements

```bash
pip install numpy scipy pandas matplotlib
```

### Run

Open and execute `researchnotefinal_02182026.ipynb` sequentially. The notebook:

1. Defines all SU(2) utilities and objective functions
2. Runs the multi-start optimization study
3. Computes ridge-side diagnostics
4. Generates Tables II and III from the paper
5. Saves results to CSV and LaTeX

To reproduce from raw data only (skip optimization):

```python
import pandas as pd
df = pd.read_csv("su2_pra_table2_results_1700.csv")
tables = make_extreme_tables(df, n=10, verbose=True)
```

---

## Key Results

| Finding | Value |
|---|---|
| Total converged runs | 11,900 |
| Interior solutions | 11,734 (98.6%) |
| Boundary-terminated | 166 (1.4%) |
| Max interior ||∇L_G|| | 34.633 (B1 small, λ=10⁻⁶) |
| Max fidelity improvement | 0.848 (flat infidelity 0.872 → geometric 0.024) |

The asymmetry in fidelity outcomes is structurally predicted by the theory: cases where geometric re-optimization yields higher infidelity show differences of at most 0.063 (noise-level variation near a shared basin), while improvements are concentrated in stress configurations with large-norm true parameters where Euclidean regularization introduces a spurious radial bias.

---

## Citation

```bibtex
@article{fulton2026referencemeasure,
  title   = {Reference-Measure Geometry in Quantum Parameter Estimation:
             When Coordinate Surrogates Produce Ridge-Side Convergence},
  author  = {Fulton, Christopher P. and Fulton, Lawrence V.},
  journal = {Physical Review A},
  year    = {2026},
  note    = {Submitted}
}
```

---

## License

Code released under the MIT License. See `LICENSE` for details.

---

## Contact

- Christopher P. Fulton — fultonc742@gmail.com  
- Lawrence V. Fulton — fultonl@bc.edu (Boston College, Applied Analytics)
