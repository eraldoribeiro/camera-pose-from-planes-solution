# Camera pose from planes

A tutorial on recovering a camera's **position and orientation** from a single
photograph of a **planar** object, by estimating a homography and then
factorizing it.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/eraldoribeiro/camera-pose-from-planes-solution/blob/main/pose_from_planes_4points.ipynb)

The worked example uses four photographs of the same rectangular object taken
from different viewpoints, and recovers where the camera was standing for each
one.

---

## The idea

When a camera photographs a **plane**, the mapping between the plane and the
image is a homography in the form of a single $3\times3$ matrix. Placing the world frame on
the plane makes every object point have $w=0$, which cancels the third column of
the rotation and collapses the $3\times4$ projection matrix into

$$\Phi \;\simeq\; \Lambda\,[\,\mathbf{r}_1 \;\; \mathbf{r}_2 \;\; \boldsymbol{\tau}\,]$$

where $\Lambda$ holds the intrinsics and $\mathbf{r}_1,\mathbf{r}_2$ are the first two columns of the rotation.

This is also a recipe for **pose**: if the camera is calibrated,
$\Lambda^{-1}\Phi$ leaves the pose, scaled by an unknown factor. Because the
columns of a rotation matrix are unit vectors, that scale can be estimated 
directly, and the rest of the pose follows.

The notebook derives the camera-pose estimation in five steps: remove the intrinsics, recover the
scale $\lambda$, fix its sign, build the rotation, and project the result onto $\mathrm{SO}(3)$.

## Notebook steps

1. **Calibrate the camera** from 41 chessboard images, giving $\Lambda$ and the
   lens distortion.
2. **Set up correspondences** between four reference corners of the model
   rectangle and the matching corners clicked in each photograph.
3. **Estimate the homography** for each image with `cv.findHomography`.
4. **Factorize each homography** into a rotation $\Omega$ and translation $\boldsymbol{\tau}$.
5. **Plot the recovered cameras** in 3-D alongside the plane they were
   photographing, so the geometry can be checked by eye.

## Running it

**On Colab** — click the badge above or use Chrome's Colab extension). The first cell installs `pytransform3d`
and `cvxopt`, and clones this repository so the image files are available.

**Locally** — clone the repository and launch the notebook from inside it, so
the relative paths to the images resolve:

```bash
git clone https://github.com/eraldoribeiro/camera-pose-from-planes-solution.git
cd camera-pose-from-planes-solution
jupyter notebook pose_from_planes_4points.ipynb
```

Requires `numpy`, `opencv-python`, `matplotlib`, `scipy`, `cvxopt` and
`pytransform3d`.

## What is in the repository

| Path | Contents |
|---|---|
| `pose_from_planes_4points.ipynb` | the tutorial |
| `calibration/` | 41 chessboard images, 640×480, for calibration |
| `dory_rectangle0..3.jpg` | four views of the planar object, 800×600 |
| `dory.jpg` | the reference pattern, 512×512 |
| `plot_image_grid.py` | helper for displaying images in a grid |

## Notes

- **The intrinsics are resolution-dependent.** The calibration images are
  640×480 and the object images are 800×600, so $\Lambda$ has to be rescaled
  before it is used on them. Skipping this leaves the focal length about 20%
  short, which tilts every recovered pose.
- **The scale has an ambiguous sign.** $\Phi$ and $-\Phi$ are the same
  homography. The notebook resolves it by requiring $\tau_z > 0$ — the object
  must be *in front of* the camera.
- **The third rotation column is computed, not measured.** A planar target
  carries no information about the plane normal, so
  $\mathbf{r}_3 = \mathbf{r}_1 \times \mathbf{r}_2$.
- **Noise breaks orthonormality.** The recovered $[\,\mathbf{r}_1\;
  \mathbf{r}_2\; \mathbf{r}_3\,]$ is close to a rotation but not exactly one, so
  it is projected onto the nearest true rotation using the SVD.
- **Drawing a camera needs the inverse pose.** The estimated pose maps *world to
  camera*; to place the camera in the world you need
  $[\,\Omega^\mathsf{T} \;\; -\Omega^\mathsf{T}\boldsymbol{\tau}\,]$. Used
  directly, every camera would sit at the origin.

## Further reading

- S. J. D. Prince, *Computer Vision: Models, Learning, and Inference* — the
  notation used throughout.
- R. Hartley and A. Zisserman, *Multiple View Geometry in Computer Vision* —
  homography estimation and pose recovery in depth.
- Z. Zhang, *A Flexible New Technique for Camera Calibration*, PAMI 2000 — the
  planar calibration method this factorization comes from.
