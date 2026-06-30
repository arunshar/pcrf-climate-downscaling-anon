# Experiments

## `synthetic_sanity_check.py` — physics-penalty mechanism validation

Pure-numpy check (no torch, no GPU, no downloads) that the three PC-RF physics
penalties actually do what the paper claims: reduce divergence error (DE),
non-negativity violations (NVR), and mass-conservation error (MCE) **without**
destroying signal fidelity.

```bash
python3 synthetic_sanity_check.py
```

Outputs `results/synthetic_metrics.md` and `results/synthetic_physics_demo.png`.

### Result (synthetic toy fields, seed 0)

| Method | DE ↓ | NVR ↓ | MCE ↓ | RMSE ↓ | Corr ↑ | SSIM ↑ |
|---|---|---|---|---|---|---|
| Ground truth | 0.0000 | 0.000 | 0.0000 | 0.0000 | 1.000 | 1.000 |
| RF-base (no physics) | 0.0081 | 0.236 | 0.1838 | 0.1397 | 0.956 | 0.518 |
| PC-RF (physics) | 0.0057 | 0.000 | 0.0000 | 0.1116 | 0.966 | 0.682 |

PC-RF cuts divergence error 28.7%, eliminates all negativity violations
(0.236 → 0), zeroes mass error, and *improves* correlation (0.956 → 0.966) and
SSIM (0.518 → 0.682). The penalties help fidelity rather than trading it away.

## ⚠️ Scope and honesty

These numbers are **synthetic** — toy divergence-free wind and Gaussian-blob
precipitation, not ERA5 or Sentinel-2. They validate the *mechanism* and
de-risk the code. They must **not** be placed in the paper's results tables.

For the paper's reported numbers, run the PyTorch pipeline in `../code/` on real
ERA5 / Sentinel-2 data (see `../code/README.md`).
