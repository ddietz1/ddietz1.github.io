---
layout: project
title: "2D physics simulator for a jack in a box"

carousel_images:
  - /assets/images/JackintheBox.png
carousel_height: 400px
carousel_width: auto
preview_gif: "/assets/images/Jackinthebox.gif"
code: "https://github.com/ddietz1/2D-Physics-Sim"
# data: "https://example.com/dataset"
tags: ["Sympy", "Python"]
date: 2025-12-16
description: "A 2D physics simulator displaying Lagrangian impact dynamics for a jack in a box."
---

## Overview

This project demonstrates a 2D rigid-body physics simulator of a jack-in-the-box system using first-principles mechanics and symbolic computation. The dynamics are derived from the Euler–Lagrange formulation, with all kinematics expressed through SE(3) frame transformations and spatial inertia matrices.

The system models both the box and a four-armed jack as coupled rigid bodies subject to gravity, external forcing, and contact constraints. Impacts between the jack arms and box walls are handled using constraint-based collision detection and momentum/energy-conserving impact equations. Symbolic expressions are generated with SymPy and converted to efficient numerical functions for real-time simulation using Runge–Kutta integration.

## Key Features

* Symbolic multibody dynamics using Euler–Lagrange equations

* SE(3) transformation-based kinematics across 13 reference frames

* Spatial inertia and energy-based modeling

* 16 contact constraints for collision detection

* Momentum/energy-preserving impact updates

* RK4 numerical integration for stable real-time animation

* Implemented in Python with SymPy and NumPy

{% include video.liquid
  path="assets/images/2DSimVideo_web.mp4"
  title="2D Simulation"
  class="img-fluid rounded z-depth-1"
  controls="true"
  autoplay="true"
  loop="true"
  muted="true"
  playsinline="true"
  width="70%"
%}

