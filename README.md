# Panorama Stitching from Scratch

A complete image-stitching pipeline, implemented step by step in a single notebook, with all results and tests already executed inside it.

## Files

| File | What it is |
|---|---|
| `Panorama_Stitching_from_Scratch.ipynb` | The deliverable. Runs on Google Colab or locally. Saved **with outputs**, so every figure, table and test result is already visible. |
| `results/panorama_*.png` | The four final panoramas (full resolution). |
| `data/` | The public image sets (OpenCV `opencv_extra` stitching test data). The notebook downloads them itself if the folder is missing. |

## Run it on Colab

1. Open https://colab.research.google.com, choose **File > Upload notebook** and pick `Panorama_Stitching_from_Scratch.ipynb`.
2. **Runtime > Run all** (a few minutes on a CPU runtime). No installs are needed: it uses NumPy, SciPy, Matplotlib and OpenCV, all preinstalled on Colab.
3. To stitch your own photos, set `USE_OWN_PHOTOS = True` in section 11 and run that cell (Colab shows an upload dialog).

Locally: `pip install numpy scipy matplotlib opencv-python jupyter`, then open the notebook from this folder.

## What is implemented (all of it in the notebook)

1. **Features**: SIFT (ORB as alternative) keypoints and descriptors.
2. **Matching**: own brute-force matcher, Lowe ratio test, mutual check.
3. **Homography**: own normalized DLT, own MSAC-RANSAC (adaptive iterations, degenerate-sample rejection, symmetric transfer error), Levenberg-Marquardt polish.
4. **Alignment**: pairwise match graph, maximum spanning tree (photos may come in any order), tree-centre reference, global least-squares refinement, focal-length estimate, automatic planar or cylindrical projection.
5. **Warping**: own bilinear inverse warp into the panorama frame.
6. **Blending and post-processing**: gain (exposure) compensation, feathering, multi-band Laplacian-pyramid blending, largest-rectangle crop of the black borders.

OpenCV is used only for the SIFT detector, Gaussian pyramids, distance transforms and I/O. `cv2.Stitcher` and `cv2.findHomography` appear only as independent baselines in the validation section.

## Results (executed run)

| Set | Photos | Projection | Panorama | Mean alignment error (global) |
|---|---|---|---|---|
| `boat` (harbour, camera pan) | 6 | cylindrical | 2744 x 601 | 0.98 px |
| `budapest` (6 scanned map tiles, 2 x 3 grid) | 6 | planar | 2044 x 987 | 1.34 px |
| `newspaper` (flat page) | 4 | planar | 1580 x 987 | 0.32 px |
| `a` (church interior) | 3 | cylindrical | 770 x 688 | 1.15 px |

## Is it working right? Validation (section 10): 32 of 32 checks pass

Each component is tested against ground truth: known homographies (exact, noisy, 30/50/70 % outliers), brute-force searches (nearest-pixel fill, largest rectangle), OpenCV's own functions as independent references (`BFMatcher`, `warpPerspective`, `findHomography`), a synthetic panorama with known geometry, shuffled order and injected exposure changes (corner error below 1.1 px, PSNR 34 dB against the true scene versus 27.6 dB for naive overlay), and real-data checks (all photos connected, correct left-to-right order from shuffled input, an unrelated photo rejected, no black pixels in the crop).

Also re-run on Python 3.12 with NumPy 1.26 / OpenCV 4.8 (a deliberately older stack than Colab's): same 32 of 32.

## Honest notes

* Pure homographies assume a rotating camera or a flat scene. Moving objects (drifting ice in `boat`) or parallax can leave faint ghosting; there is no graph-cut seam finder.
* The global refinement adjusts homographies only (no full bundle adjustment with shared intrinsics), so 360 degree panoramas are out of scope.
* RANSAC is randomised; on one `boat` edge a single seed settles about 2 % above OpenCV's cost, so that test compares the median of 5 seeds.
