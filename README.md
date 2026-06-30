# Physics-Constrained Rectified Flow (PC-RF) for Climate Downscaling

> Anonymized copy for double-blind review (AIAS+ 2026). Author and citation details
> are omitted.

## Overview
PC-RF is a conditional rectified-flow downscaler that enforces atmospheric conservation
laws in both training and inference. Modern diffusion and flow-matching generators produce
sharp climate fields that nonetheless violate the physics encoded in their own coarse
conditioning. PC-RF adds three differentiable physics penalties during training and a
serving-time projection that makes the hard constraints exact. This repository reproduces
the paper's synthetic and ERA5 results.

## Key Contributions
- Physics penalties in the flow objective: divergence-free wind, non-negative precipitation,
  and domain mass conservation, on a one-step clean-field estimate.
- Inference-time projection that makes non-negativity and mass balance exact for any model.
- Physics-validity metrics: Divergence Error (DE), Non-negativity Violation Rate (NVR), and
  Mass-Conservation Error (MCE).
- A measured separation of what training penalties buy and what the projection guarantees.

## Requirements
Python (>= 3.9) with `numpy`, `torch`, and `matplotlib`.

## Reproducing the Results

### Synthetic mechanism check (CPU)
```
python experiments/synthetic_sanity_check.py
```
Reproduces Table 1: PC-RF drives the non-negativity violation rate and mass-conservation
error of an unconstrained baseline to zero while improving reconstruction.

### Real ERA5 downscaling (GPU)
The ERA5 pipeline is in `src/pcrf/`: download a year of ERA5 with `download_data.py`, then
train and evaluate with `train.py` and `evaluate.py`. Saved metrics are in
`results/metrics.json`.

## Results
Synthetic (PC-RF vs an unconstrained baseline): NVR 0.236 to 0.000, MCE 0.184 to 0.000,
RMSE 0.140 to 0.112, SSIM 0.52 to 0.68, divergence about 30% lower.

Real single-year ERA5 (one GPU): the projection makes NVR and MCE exactly zero for both
models; PC-RF keeps a small perceptual edge (SSIM 0.914 vs 0.912, PSNR 26.59 vs 26.39,
CRPS 1.149 vs 1.188).

## Repository Structure
```text
.
|-- paper/                      # the AIAS+ paper (EasyChair), figures, references
|-- src/pcrf/                   # the PC-RF method: flow, physics penalties, model, metrics
|-- experiments/                # synthetic mechanism check
|-- results/                    # metrics.json and the result tables
`-- requirements.txt
```

## Citation
Omitted for double-blind review.
