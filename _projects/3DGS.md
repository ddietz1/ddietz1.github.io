---
layout: project
title: "Next-Best-View 3D Gaussian Splatting with an Autonomous Mobile Robot"

preview_gif: "/assets/images/SLAM_Final.gif"
code: "https://github.com/ddietz1/3DGS-Optimization-with-Stretch-3"

tags: ["ROS 2", "Python", "3DGS", "Hello Robot", "PyTorch", "NAV2"]
date: 2026-6-1
description: "A closed-loop autonomous exploration system where a mobile robot iteratively selects, navigates to, and captures its own next-best viewpoints, continuously improving a live 3D Gaussian Splatting reconstruction of its environment without any human guidance."
---

{% include video.liquid
  path="assets/videos/Stretch_3_Sim_Initial_Capture.mp4"
  title="3DGS Optimization with Stretch 3"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
  width="20%"
%}

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

<div style="text-align:center;">
  <img src="/assets/images/Stretch_3_Image.png" width="200"><br>
  <em>Stretch 3</em>
</div>

## Results & Validation

Rendering the same camera pose from the model checkpoint immediately before and after that pose was captured and trained on shows the reconstruction resolving new geometry.

| Round | Before (dB) | After (dB) | Δ |
|---|---|---|---|
| 0 | 11.61 | 20.73 | +9.12 |
| 1 | 12.45 | 20.17 | +7.72 |
| 2 | 11.84 | 22.17 | +10.33 |
| 3 | 12.95 | 21.36 | +8.41 |

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
{% include video.liquid
  path="assets/videos/captured_pose_video_rotated/bad_pose_evolve.mp4"
  title="The top scoring pose being rendered as the model updates"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
  width="40%"
%}

## Citations
This project's Shannon-MI viewpoint scoring is a plain-PyTorch reimplementation based on the GauSS-MI approach.
@article{xie2025gaussmi,
  title={GauSS-MI: Gaussian Splatting Shannon Mutual Information for Active 3D Reconstruction},
  author={Xie, Yuhan and Cai, Yixi and Zhang, Yinqiang and Yang, Lei and Pan, Jia},
  journal={arXiv preprint arXiv:2504.21067},
  year={2025}
}