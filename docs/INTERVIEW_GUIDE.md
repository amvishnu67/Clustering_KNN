# Interview Guide

This document is the short narrative to use when walking a hiring manager through the repository.

## Recommended Story

Frame the work as a two-stage CV pipeline:

1. **Task 1** solves structured 2D detection and correspondence recovery.
2. **Task 2** tests whether those correspondences are geometrically consistent under the provided calibration assumptions.

That is the right level for a hiring discussion. It shows both implementation and scientific validation.

## Task 1 Talking Points

Use [notebooks/Mercedes.ipynb](../notebooks/Mercedes.ipynb) and the generated PNGs in `notebooks/`.

### What to say

- The projector image is a clean binary-like pattern, so a lower threshold (`50`) preserves the ring boundaries.
- The camera image contains glow, blur, and scene background, so a higher threshold (`180`) suppresses non-pattern pixels.
- Area filtering was used to keep connected components inside the expected blob-size range for this resolution.
- The expected prior was 40 circles, so the threshold and area settings were tuned to recover exactly 40 valid centroids.

### Quantitative result

- 40/40 valid matches
- mean reprojection error: `3.219 px`
- max reprojection error: `6.145 px`

That is a strong Task 1 result and should be shown clearly.

## Task 2 Talking Points

Use [notebooks/Mercedes_task2_signcheck.ipynb](../notebooks/Mercedes_task2_signcheck.ipynb).

### Core concept

The provided intrinsics do not uniquely resolve whether the local sensor Y axis should be treated as "up" or "down" in 3D back-projection. That ambiguity matters because it changes the ray direction and therefore the triangulated 3D geometry.

### Theory-aligned convention

For interview use, the easiest convention to defend is:

- use `T_C_from_P` directly
- interpret camera/projector axes as `+x right`, `+y up`, `+z forward`
- map image `v`-down to 3D `+y`-up using a Y-flip in back-projection

The corresponding equation is:

\[
\mathbf{d}_c \propto
\mathbf{S}_y \,\mathbf{K}_C^{-1}
\begin{bmatrix}
u_c\\
v_c\\
1
\end{bmatrix},
\qquad
\mathbf{S}_y=
\begin{bmatrix}
1&0&0\\
0&-1&0\\
0&0&1
\end{bmatrix}
\]

The same form is applied to the projector ray before rotating it into the camera frame.

### What to say about the result

The fictitious points are the cleanest presentation case because they are explicitly allowed by the assignment.

Result under the theory-aligned convention:

- `s>=0: 2/3`
- `l>=0: 2/3`
- `Z>0: 2/3`
- ray-gap min/mean/max: `0.180876 / 0.272489 / 0.425231 m`

Per-point values:

- `P1`: `X1 = [0.05235747, -0.09522955, 2.5594368]`, `s1 = 2.5596`, `l1 = 2.3992`, `gap1 = 0.211361`
- `P2`: `X2 = [-0.5884177, 0.04060404, -11.78608321]`, `s2 = -11.8005`, `l2 = -12.0359`, `gap2 = 0.180876`
- `P3`: `X3 = [0.24264679, -0.16233443, 2.72057266]`, `s3 = 2.7279`, `l3 = 2.5255`, `gap3 = 0.425231`

Plane fit:

- normal `[-0.33719266, -0.94141559, 0.00614748]`
- equation `-0.337193 X -0.941416 Y +0.006147 Z -0.087730 = 0`

### How to explain the failure mode

One reconstructed point has negative depth and large ray-gap values remain. That should not be hidden. The correct explanation is:

- the notebook is doing the right validation
- the chosen convention is theoretically aligned with the written assignment
- the data still reveals inconsistency between theory and observation

That is not a weakness in presentation. It shows that you know how to diagnose calibration-model mismatch instead of blindly trusting a pipeline.

## Empirical Cross-Check

The real CSV-matched points behaved better under:

- `inv(T_C_from_P)`
- `cam_y_up=False`
- `proj_y_up=False`

For labels `2, 15, 24`, this gave:

- `s>=0: 3/3`
- `l>=0: 3/3`
- `Z>0: 3/3`
- ray-gap min/mean/max: `0.016998 / 0.050875 / 0.089626 m`

This is worth mentioning only as a validation note:

- theory-aligned convention is easier to defend conceptually
- inverse-transform convention is better aligned with the real recovered correspondences

That distinction is exactly the kind of judgment hiring managers expect from a strong CV engineer.
