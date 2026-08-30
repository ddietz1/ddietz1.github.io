---
layout: project
title: "Next-Best-View 3D Gaussian Splatting with an Autonomous Mobile Robot"

preview_gif: "assets/videos/3DGS_Demo.gif"
code: "https://github.com/ddietz1/3DGS-Optimization-with-Stretch-3"

tags: ["ROS 2", "Python", "3DGS", "Hello Robot", "PyTorch", "NAV2"]
date: 2026-6-1
description: "A closed-loop autonomous exploration system where a mobile robot iteratively selects, navigates to, and captures its own next-best viewpoints, continuously improving a live 3D Gaussian Splatting reconstruction of its environment without any human guidance."
---

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; margin-top: 1rem;">
  <iframe 
    src="https://www.youtube.com/embed/giVTByQmJT4"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
    frameborder="0" allowfullscreen>
  </iframe>
</div>

## Overview

A ROS 2 + PyTorch system that closes the loop between robotic exploration and 3D reconstruction: a Hello Robot Stretch 3 autonomously decides where to look next, physically navigates there, and feeds each new observation back into a live 3D Gaussian Splatting model, repeating until the reconstruction stops meaningfully improving. Viewpoint selection is driven by a Shannon mutual-information scoring function over the Gaussian scene representation, explicitly choosing the views that resolve the most uncertainty.

## System Architecture

<div style="text-align:center;">
  <img src="/assets/images/Final_Project_flowchart.png" width="200"><br>
  <em>System Architecture</em>
</div>

## Hardware

### Intel RealSense D435i
The onboard RGB-D camera mounted on the Stretch 3's head, providing the color images and depth maps used both for live capture and for seeding each new region of the Gaussian point cloud.

### Hello Robot Stretch 3
The mobile manipulator platform. Its head pan/tilt joints aim the D435 at a requested 6-DOF pose, and its mobile base (driven via Nav2) carries it to the standoff position for each next-best-view candidate before the head IK takes over for final aiming.

<div style="max-width: 500px; margin: 0 auto; text-align:center;">
  <img src="/assets/images/Stretch_3_Image.png"><br>
  <em>Stretch 3</em>
</div>

## Results & Validation

To validate that Shannon mutual-information scoring is actually driving the exploration, I ran two identical 11-round pipelines(one that uses the scoring mechanism to determine the next capture pose and another that randomly samples from the candidate poses to determine the next capture pose), holding everything else fixed (same initial images, same holdout poses, same training budget). Two independent metrics tell the same story. First, the model's own top-candidate uncertainty score, which the informed run should be actively paying down, fell 82.7% over 11 rounds under scored selection, but rose 75.8% under random selection, confirming that without targeted selection, the scene's worst-covered regions are never visited. Second, real image quality follows suit: on 3 fixed held-out views never trained on by either run, scored selection held a consistent ~0.3 dB PSNR edge across most rounds (mean 17.90 dB vs. 17.65 dB), and at the newly-captured pose itself, an informed round-0 pick resolved more than twice the reconstruction error of a random one (+10.40 dB vs. +4.52 dB) — a gap that narrows by round 10 (+5.99 dB vs. +5.59 dB) as overall scene coverage saturates for both strategies.

<div style="max-width: 500px; margin: 0 auto; text-align:center;">
  <img src="/assets/images/scored_vs_random/held_out_view_quality.png"><br>
  <em>PSNR values for scored versus random NBV selection</em>
</div>

<div style="max-width: 500px; margin: 0 auto; text-align:center;">
  <img src="/assets/images/scored_vs_random/remaining_scene_uncertainty.png"><br>
  <em>Uncertainty values for scored versus random NBV selection</em>
</div>

Rendering the same camera pose from the model checkpoint immediately before and after that pose was captured and trained on shows the reconstruction resolving new geometry.

**Scored selection:**

| Round | Before (dB) | After (dB) | Δ |
|:---:|:---:|:---:|:---:|
| 0 | 12.74 | 23.14 | +10.39 |
| 1 | 13.28 | 21.33 | +8.04 |
| 2 | 10.94 | 20.45 | +9.51 |
| 3 | 15.39 | 19.99 | +4.59 |
| 4 | 14.66 | 23.91 | +9.25 |
| 5 | 13.51 | 19.44 | +5.94 |
| 6 | 15.44 | 19.43 | +3.99 |
| 7 | 13.65 | 21.92 | +8.28 |
| 8 | 10.94 | 23.36 | +12.42 |
| 9 | 15.35 | 18.74 | +3.40 |
| 10 | 15.30 | 21.29 | +5.99 |
| **Mean** | **13.75** | **21.18** | **+7.44** |

**Random baseline:**

| Round | Before (dB) | After (dB) | Δ |
|:---:|:---:|:---:|:---:|
| 0 | 12.94 | 17.46 | +4.53 |
| 1 | 12.60 | 17.85 | +5.25 |
| 2 | 14.40 | 17.70 | +3.30 |
| 3 | 13.05 | 19.21 | +6.16 |
| 4 | 14.61 | 23.02 | +8.41 |
| 5 | 16.79 | 19.17 | +2.38 |
| 6 | 14.65 | 18.81 | +4.17 |
| 7 | 14.76 | 16.97 | +2.21 |
| 8 | 14.51 | 20.65 | +6.14 |
| 9 | 13.17 | 21.07 | +7.89 |
| 10 | 13.78 | 19.37 | +5.59 |
| **Mean** | **14.11** | **19.21** | **+5.09** |

The consistent spike in Peak Signal to Noise Ratio(PSNR) across multiple rounds shows the uncertain geometry in the scene being resolved. 

## Media

### 3DGS Renderings

The first image below is a rendering of the top scoring candidate pose rendered from the initial model. The second image was rendered from the next model checkpoint, after an image at that pose was captured by the robot and the model trained with that image added to the dataset. The third image is the actual RGB photo captured by the onboard D435i camera. Plainly the additional capture was able to resolve much of the uncertainty that the initial model failed to do.

<div style="display:flex; justify-content:center; gap:16px; flex-wrap:wrap;">
  <div style="text-align:center;">
    <img src="/assets/round_pose_compare/round0_before.png" width="200"><br>
    <em>Rendered pose - initial model</em>
  </div>
  <div style="text-align:center;">
    <img src="/assets/round_pose_compare/round0_after.png" width="200"><br>
    <em>Rendered pose - updated model</em>
  </div>
  <div style="text-align:center;">
    <img src="/assets/round_pose_compare/round0_groundtruth.png" width="200"><br>
    <em>Actual RGB image</em>
  </div>
</div>

## Video demonstrations

### Top Scoring Pose After Model Retraining
<div style="max-width: 40%; margin: 0 auto;">
{% include video.liquid
  path="assets/videos/captured_pose_video_rotated/bad_pose_evolve.mp4"
  title="The top scoring pose being rendered as the model updates"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
%}
</div>

### Initial versus Optimized Model
<div style="max-width: 50%; margin: 0 auto;">
{% include video.liquid
  path="assets/videos/flythrough/rotated/side_by_side.mp4"
  title="A flythrough showing the intially created model side by side with the optimized model"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
%}
</div>

## Citations
This project's Shannon-MI viewpoint scoring is a plain-PyTorch reimplementation based on the GauSS-MI approach.
@article{xie2025gaussmi,
  title={GauSS-MI: Gaussian Splatting Shannon Mutual Information for Active 3D Reconstruction},
  author={Xie, Yuhan and Cai, Yixi and Zhang, Yinqiang and Yang, Lei and Pan, Jia},
  journal={arXiv preprint arXiv:2504.21067},
  year={2025}
}