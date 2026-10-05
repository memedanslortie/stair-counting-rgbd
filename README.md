# Stair Step Counting from RGB-D Images

> Master's project, Image Analysis course (Camille Kurtz), M1 Vision & Machine Intelligence,
> Université Paris Cité (2025). Team of 2.

Estimates the **number of steps in a staircase** from a grayscale image and its depth map,
using **only classical image processing**: no machine learning, fully deterministic.

## Pipeline

```
image ─► Gaussian blur + CLAHE ─► Sobel edges ─► PCA on edge points ─► main stair axis
                                                                         │
depth map ─────────────────────────────────────► depth profile sampled along the axis
                                                                         │
                               Gaussian smoothing + adaptive-prominence peak detection
                                                                         │
                                                                   step count
```

1. **Preprocessing**: Gaussian blur against noise, CLAHE for contrast under uneven lighting.
2. **Edge detection**: thresholded Sobel gradients.
3. **PCA** on edge coordinates gives the dominant orientation of the staircase.
4. **Depth profile**: the depth map is sampled along that axis, so each step shows up as a
   transition in the 1-D signal.
5. **Step detection**: the profile is smoothed, then `scipy.signal.find_peaks` runs with an
   adaptive prominence threshold.

A Hough-transform line detector was tried first. It was dropped because it was too
sensitive to per-image parameters (details in the [full report](docs/REPORT_FR.md)).

## Results

Evaluated on a held-out test set of **66 images**, stratified by visual difficulty.

| Difficulty | Images | MAE (steps) |
|---|:-:|:-:|
| Easy | 46 | 2.78 |
| Medium | 12 | 3.92 |
| Hard | 8 | 4.88 |
| **Overall** | **66** | **3.24** |

For medium-length staircases (32 of the 66 images), the MAE drops to **1.94 steps**. The error grows on very long staircases and on images that break the
method's assumptions (frontal view, straight staircase, clean depth map):

<p align="center"><img src="results/scatter_mae_vs_steps.png" width="560" alt="Absolute error vs. number of steps"></p>

## Build & run

Requires OpenCV ≥ 4 (C++) and Python 3 with `numpy` and `scipy`.

```bash
make                 # builds ./stair_detector
./stair_detector     # runs on data/img + data/depth, writes metrics to results/
python scripts/plot.py
```

## Repository layout

```
src/, include/   C++ pipeline (preprocessing, PCA, depth-profile extraction, evaluation)
peak.py          Peak detection on the depth profile
scripts/         Stratified val/test split, metrics, plots
data/            Images, depth maps, annotations
results/         Metrics and figures
docs/            Detailed report (French)
```

## My contribution (Auguste Calmanovic-Plescoff)

- **Depth-profile extraction**: vertical and rotated profile sampling on the depth map.
- **Stratified evaluation**: MAE broken down by difficulty and by step-count category,
  plus the analysis plots.
- Documentation and the written report.

Co-author: [Yassine Fekih](https://github.com/yassinefkh). He built the core
preprocessing → PCA pipeline and the earlier Hough-transform experiments.
