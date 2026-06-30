# PC-RF experiment code

Reference implementation of **Physics-Constrained Rectified Flow** (PC-RF) for
the AIAS 2026 paper. Runnable, modular PyTorch.

## Status: validated

The full pipeline is smoke-tested end-to-end on CPU (torch 2.12, Python 3.12):
dataset schema, model forward, training loss + backward through all three
physics penalties, ODE sampling, the inference-time physics projection
(drives NVR and MCE to exactly 0), and every metric. See `smoke_test.py`.

> **These are not the paper's results.** The synthetic dataset exists only to
> exercise the code. Real numbers for the paper require ERA5 + Sentinel-2 data
> on a GPU (fill in the stubs in `data.py`).

## Files

| File | Purpose |
|---|---|
| `physics.py`  | Divergence / non-negativity / mass penalties + inference projection |
| `model.py`    | Conditional U-Net velocity field, FiLM, Sentinel-2 encoder |
| `flow.py`     | Rectified-flow training loss + ODE sampling |
| `data.py`     | `SyntheticDownscaleDataset` (runs anywhere) + ERA5/Sentinel-2 stubs |
| `metrics.py`  | PSNR, SSIM, CRPS, DE, NVR, MCE |
| `train.py`    | Training entry point (argparse) |
| `evaluate.py` | Evaluation entry point -> metrics table |
| `smoke_test.py` | End-to-end pipeline check (CPU, seconds) |

## Setup

```bash
python3.12 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

For a real GPU run, install the CUDA build of torch from pytorch.org.

## Quick start (synthetic, CPU)

```bash
python smoke_test.py                       # validate everything runs
python train.py --data synthetic --epochs 2 --batch 8 --fine 64 --base 32 \
    --out /tmp/pcrf_ckpt
python evaluate.py --ckpt /tmp/pcrf_ckpt/pcrf_last.pt --data synthetic \
    --fine 64 --steps 20 --ensemble 8
```

## Tests

Function-wide unit tests (physics, model, flow, metrics, data) plus an
end-to-end pipeline test live in `tests/`:

```bash
python -m pytest tests/ -q        # 26 tests, CPU, seconds
```

The end-to-end test trains briefly, samples, and asserts the inference projection
drives NVR to 0 and MCE below 1e-3, the paper's serving-time guarantee.

## Reproducing the paper (real data)

1. **Point the loaders at your data.** `ERA5Dataset` and `Sentinel2Dataset` in
   `data.py` are implemented (xarray / rioxarray) and wired into `train.py`. For
   ERA5 pass `--data era5 --coarse coarse.nc --target target.nc`; for Sentinel-2,
   `--data sentinel2 --scenes scenes.json`. Both return the synthetic dataset's
   dict schema, so the rest of the pipeline is unchanged. (The ERA5 path is tested
   end to end against a toy NetCDF; see `tests/test_data_ingestion.py`.)
2. **Train PC-RF and the ablations:**
   ```bash
   # full PC-RF
   python train.py --data era5 --coarse /data/era5/coarse.nc \
       --target /data/era5/target.nc --epochs 200 --batch 16 \
       --fine 128 --base 64 --l_div 1.0 --l_nn 1.0 --l_mass 1.0 --out ckpt/pcrf
   # ablations
   python train.py ... --l_div 0      --out ckpt/no_div
   python train.py ... --l_mass 0     --out ckpt/no_mass
   python train.py ... --no_physics   --out ckpt/rf_base
   ```
3. **Evaluate each checkpoint** with `evaluate.py` and assemble Tables 1, 2, 3.
4. Tune `lambda_1..3` (`--l_div/--l_nn/--l_mass`) on the validation set; record
   the chosen values in the paper's Appendix A.

## Mapping to the paper

- Eq. 2 (divergence-free), Eq. 3 (non-negativity), Eq. 4 (mass) -> `physics.py`
- Eq. 1 (rectified-flow objective), Eq. 5 (combined loss) -> `flow.py`
- FiLM fusion + U-Net (Sec 3.4-3.5, Fig. 1) -> `model.py`
- DE / NVR / MCE metrics (Sec 4) -> `metrics.py`
