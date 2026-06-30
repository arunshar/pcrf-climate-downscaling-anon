# Synthetic physics-penalty sanity check (NOT ERA5 results)

Pure-numpy mechanism check. Demonstrates that the PC-RF physics
penalties reduce DE/NVR/MCE without destroying signal fidelity.
These numbers are SYNTHETIC and must not appear in the paper's
ERA5/Sentinel-2 tables.

| Method | DE down | NVR down | MCE down | RMSE down | Corr up | SSIM up |
|---|---|---|---|---|---|---|
| Ground truth | 0.0000 | 0.000 | 0.0000 | 0.0000 | 1.000 | 1.000 |
| RF-base (no physics) | 0.0081 | 0.236 | 0.1838 | 0.1397 | 0.956 | 0.518 |
| PC-RF (physics) | 0.0057 | 0.000 | 0.0000 | 0.1116 | 0.966 | 0.682 |

**PC-RF vs RF-base (synthetic):** divergence error 28.7% lower, NVR 0.236 to 0.000, MCE 100.0% lower, correlation 0.956 to 0.966.
