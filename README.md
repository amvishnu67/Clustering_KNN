# Mercedes Projection-to-Camera Geometry Case Study

This repository packages a compact computer-vision case study around projected circle detection, 2D homography matching, and 3D triangulation analysis. The goal is to present a complete structured-light reconstruction workflow with the reasoning, validation, and failure analysis needed to understand the result end-to-end.

## What This Repository Demonstrates

1. Detecting a structured projected pattern in both projector-space and camera-space images.
2. Recovering 40 matched 2D correspondences with a homography-based pipeline.
3. Testing camera/projector ray back-projection conventions for 3D reconstruction.
4. Triangulating 3D points and fitting a wall plane.
5. Comparing theoretical consistency against empirical consistency when calibration assumptions disagree with observed data.

## Repository Structure

- [notebooks/Mercedes.ipynb](./notebooks/Mercedes.ipynb): Task 1 pipeline for blob detection, ordering, homography matching, and output generation.
- [notebooks/Mercedes_task2_signcheck.ipynb](./notebooks/Mercedes_task2_signcheck.ipynb): Task 2 pipeline for sign-convention-aware triangulation and plane fitting using the fictitious points from the assignment.
- [notebooks/matched_points_homography_refined_3848x2168.csv](./notebooks/matched_points_homography_refined_3848x2168.csv): 40 matched projector-to-camera correspondences from Task 1.
- `notebooks/00_*.png` to `07_*.png`: Task 1 visual artifacts used for debugging and presentation.
- [docs/CASE_STUDY_NOTES.md](./docs/CASE_STUDY_NOTES.md): structured explanation of decisions, assumptions, and interpretation of the final results.
- `data/Aufgabe_2_projection_circles.png`, `data/Aufgabe_2_photo_circles.png`: the two images used for the Mercedes CV case study.

## Task 1 Summary

Task 1 detects the ring pattern in a projector image and a captured camera image, then establishes 40 correspondences.

Key outputs:

- 40 projector centroids detected
- 40 camera centroids detected
- 40/40 unique homography-based matches
- mean reprojection error: `3.219 px`
- max reprojection error: `6.145 px`

Key implementation choices:

- `PROJECTOR_THRESH = 50`
- `CAMERA_THRESH = 180`
- `PROJ_AREA_MIN, PROJ_AREA_MAX = 40, 300`
- `CAM_AREA_MIN, CAM_AREA_MAX = 2200, 7500`

These values were tuned empirically to keep the expected 40 valid connected components while rejecting small noise fragments and merged bright regions.

## Key Visual Results

These figures are the main visual evidence for the Task 1 pipeline and match the material used in the presentation.

### 1. Camera ROI

The first step is to constrain detection to the wall region where the projected pattern appears. This reduces false detections from the surrounding dark scene and lighting fixtures.

![Camera ROI used for circle detection](./notebooks/00_camera_with_roi.png)

### 2. Indexed Projector Pattern

The clean projector-space detections are ordered row-by-row. This gives a stable indexing scheme that is later used for homography seeding and correspondence reporting.

![Indexed projector detections](./notebooks/03_projector_indexed.png)

### 3. Predicted vs Matched Camera Points

This figure is the most important Task 1 validation plot. Red points are camera positions predicted by the homography, yellow circles are the detected camera centroids, and blue segments are residual vectors. Short residuals indicate that the geometric match is stable.

![Predicted camera points versus matched detections](./notebooks/05_camera_pred_vs_matched.png)

### 4. Reprojection Error Trend

This plot summarizes the per-point residual after matching. It gives a compact quality check over all 40 correspondences and makes it easy to show that the errors stay within a tight pixel range.

![Per-point reprojection error](./notebooks/06_reprojection_error_plot.png)

### 5. Reprojection Error Distribution

The histogram complements the per-point plot by showing the overall spread of matching error. In this case the residuals cluster in a narrow band, which supports the reliability of the 2D matching stage.

![Histogram of reprojection errors](./notebooks/07_reprojection_error_hist.png)

## Task 2 Summary

Task 2 investigates 3D reconstruction from matched projector/camera points under different sign and transform conventions.

Two interpretations were evaluated:

1. **Theory-aligned assignment convention**
   - use `T_C_from_P` directly
   - use `+x right`, `+y up`, `+z forward`
   - implement `y_up=True` in back-projection

2. **Empirically best reconstruction convention for real matched labels**
   - use `inv(T_C_from_P)`
   - use `y_down` back-projection (`y_up=False`)

The repository keeps the first convention as the main reported notebook because it aligns best with the provided assignment statement and is the cleanest theoretical interpretation. The second convention is documented in the case study notes as the empirically better fit for the real matched CSV points.

### Main Reported Task 2 Result

From [notebooks/Mercedes_task2_signcheck.ipynb](./notebooks/Mercedes_task2_signcheck.ipynb), using the fictitious point pairs and the theory-aligned convention:

- `s>=0: 2/3`
- `l>=0: 2/3`
- `Z>0: 2/3`
- ray-gap min/mean/max: `0.180876 / 0.272489 / 0.425231 m`
- fitted plane:
  - normal `[-0.33719266, -0.94141559, 0.00614748]`
  - equation `-0.337193 X -0.941416 Y +0.006147 Z -0.087730 = 0`

Per-point values:

- `P1`: `X1 = [0.05235747, -0.09522955, 2.5594368]`, `s1 = 2.5596`, `l1 = 2.3992`, `gap1 = 0.211361`
- `P2`: `X2 = [-0.5884177, 0.04060404, -11.78608321]`, `s2 = -11.8005`, `l2 = -12.0359`, `gap2 = 0.180876`
- `P3`: `X3 = [0.24264679, -0.16233443, 2.72057266]`, `s3 = 2.7279`, `l3 = 2.5255`, `gap3 = 0.425231`

## How To Run

Use the `iot_ts` environment that was already used during development:

```powershell
conda activate iot_ts
```

Run Task 1:

```powershell
d:\anaconda\envs\iot_ts\python.exe -m jupyter nbconvert --to notebook --execute --inplace notebooks/Mercedes.ipynb
```

Run Task 2:

```powershell
d:\anaconda\envs\iot_ts\python.exe -m jupyter nbconvert --to notebook --execute --inplace notebooks/Mercedes_task2_signcheck.ipynb
```

## Engineering Value

This project shows more than a successful output image:

- structured feature extraction from noisy imagery
- practical homography matching
- geometric reasoning about intrinsics, extrinsics, and sign conventions
- validation using reprojection error, ray parameters, positive depth, and ray-gap diagnostics
- willingness to document when the "theoretically clean" convention and the "empirically best" convention diverge

That combination is more representative of real CV engineering work than a notebook that only produces a final number.
