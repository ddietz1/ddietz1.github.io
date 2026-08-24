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
  path="assets/images/SLAM_UNKNOWN_DATA_SIMULATION.mp4"
  title="EKF SLAM demonstration"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
  width="40%"
%}

## Overview

A ROS 2 + PyTorch system that closes the loop between robotic exploration and 3D reconstruction: a Hello Robot Stretch 3 autonomously decides where to look next, physically navigates there, and feeds each new observation back into a live 3D Gaussian Splatting model, repeating until the reconstruction stops meaningfully improving. Viewpoint selection is driven by a Shannon mutual-information scoring function over the Gaussian scene representation, explicitly choosing the views that resolve the most uncertainty.

## Hardware

### Intel RealSense D435i
The onboard RGB-D camera mounted on the Stretch 3's head, providing the color images and depth maps used both for live capture and for seeding each new region of the Gaussian point cloud.

### Hello Robot Stretch 3
The mobile manipulator platform. Its head pan/tilt joints aim the D435 at a requested 6-DOF pose, and its mobile base (driven via Nav2) carries it to the standoff position for each next-best-view candidate before the head IK takes over for final aiming.

## Software Architecture

### ROS2 Nodes

- **`robot_control`** — Contains the navigation and robot control methods that enable the Stretch 3 to navigate to a given 6-DOF pose in the map frame. Contains services for navigating a given list of waypoints for initial capture, and for navigating to specific 6-DOF poses sent from the GPU.
- **`candidate_generation`** (node: `candidate_generator.py`) — Samples the local costmap for random, Nav2-reachable 6-DOF poses in the map frame, discarding unreachable/infeasible ones, and writes the feasible poses to JSON files for the GPU-side NBV scoring pipeline to pull.

### GPU-Side Pipeline

3D Gaussian Splatting model training and viewpoint scoring both run on a separate GPU workstation. The following Python scripts drive that half of the system:

- **`gpu_main_loop.py`** — Orchestrates the entire pipeline from the GPU. Polls the robot's capture directory until the initial waypoint capture is complete, then trains the initial model via `ns_train_patched.py`. From there it repeatedly scores the currently-pulled candidate pool by Shannon Mutual Information, sends the top 10 poses back to the robot, waits for the resulting new capture, and resume-trains the model with it incorporated — repeating until `check_convergence.py` signals the reconstruction has stopped meaningfully improving.
- **`gpu_candidate_puller.py`** — Runs continuously in the background (started automatically by `gpu_main_loop.py`), pulling newly-written candidate JSON files from the robot for scoring.
- **`build_transforms_from_poses.py`** — Given a list of images and their poses in the robot's map frame, builds the `transforms.json` and seed `sparse_pc.ply` point cloud directly from the robot's own AMCL poses — standing in for the COLMAP step NerfStudio normally uses.
- **`ns_train_patched.py`** — A patched drop-in replacement for NerfStudio's `ns-train`. Lets a training run resume from a checkpoint and correctly continue optimizing on an updated dataset, including injecting newly-seen depth-seeded points into the live model and optimizer state — a real nerfstudio bug where `--load-dir` resume silently discards new points otherwise.
- **`viewpoint_scoring.py`** — A plain-PyTorch reimplementation of Shannon Mutual Information viewpoint scoring, following the GauSS-MI approach<!-- ⚠️ full citation still needed — same placeholder gap as your README's Citations section -->. Maintains a per-Gaussian-primitive reliability tensor, updated from photometric residuals over training views, and scores candidate poses by their expected information gain.
- **`check_convergence.py`** — Decides whether the active-view loop should keep going or stop, using two self-calibrating criteria (score relative to the first round's max, and diminishing returns over recent rounds) instead of a fixed, scene-specific threshold.
- **`robot_capture_next_view.py`** — Runs *on the robot*, not the GPU. Watches for newly-scored candidates and drives to/captures the top-ranked reachable pose via `robot_control`'s pose service.

## Results & Validation

<!-- ⚠️ New section — built from a real live_run18 investigation (Aug 23-24 2026). Numbers below are verified; swap in your own chart images from
     /home/derek-dietz/Final_Project/results/live_run18_before_after_report.html (or the published artifact) once you've picked the ones you want. -->

Rendering the same camera pose from the model checkpoint immediately before and after that pose was captured and trained on shows the reconstruction genuinely resolving new geometry, not just an abstract score changing:

| Round | Before (dB) | After (dB) | Δ |
|---|---|---|---|
| 0 | 11.61 | 20.73 | +9.12 |
| 1 | 12.45 | 20.17 | +7.72 |
| 2 | 11.84 | 22.17 | +10.33 |
| 3 | 12.95 | 21.36 | +8.41 |

Because every visited pose is immediately added to the training set, that comparison alone measures memorization more than generalization. A stricter test: track a pose that stays genuinely untrained-on for several rounds. Its PSNR stayed flat and noisy (10.1–10.9 dB) for four full rounds while its SMI-ranked priority climbed from 40th to 1st out of a growing candidate pool — then jumped **+9.3 dB the moment it was actually visited**. That's direct evidence the scoring was tracking real priority the whole time, independent of whether training itself was working.

**Bug found along the way:** chasing down why one high-priority pose sat unvisited for an extra round surfaced a real issue in the crash-recovery path — resuming a mid-round crash could pick up a stale, already-superseded capture instead of waiting for the robot to visit the freshly-rescored top candidate. Confirmed via rank/score comparison against the same round's own rescoring (the stale capture ranked 3rd of 442; the correct target was 1st). <!-- ⚠️ mention the fix once it's actually implemented, not just diagnosed -->

<!-- ⚠️ Not yet run — from your own README's Data section wishlist: -->
*Planned: SMI-driven selection vs. an equal-budget random-selection baseline, scored against the same held-out-pose methodology above.*

## Media

### 3DGS Renderings

The first image below is a rendering of the top scoring candidate pose rendered from the initial model. The second image was rendered from the next model checkpoint, after an image at that pose was captured by the robot and the model trained with that image added to the dataset. The third image is the actual RGB photo captured by the onboard D435i camera. Plainly the additional capture was able to resolve much of the uncertainty that the initial model failed to do.

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

## Video demonstrations

### Physical Driving
{% include video.liquid
  path="assets/images/TurtleDriveBot.mp4"
  title="Robot demonstration"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
  width="40%"
%}