---
layout: project
title: "Rapidly Exploring Random Tree"

carousel_images:
  - /assets/images/RRT.png
carousel_height: 400px
carousel_width: auto
preview_gif: "/assets/images/RRT.png"
code: "https://github.com/ddietz1/RRT"
# data: "https://example.com/dataset"
tags: ["Python"]
date: 2025-09-12
description: "A versatile, distributed platform using ROS for testing and validating a wide variety of multi-agent control algorithms."
---

## Overview

This program creates a simple Rapidly-Exploring Random Tree(RRT) using a Python program. The program starts with a random start and end point within a given domain, then iteratively moves a set amount in the direction of a new randomly generated point. Randomly generated circle obstacles are created in the program, with the number and size of the obstacles able to be easily changed. At each step, the algorithm checks if there are any obstacles between the most recent node and the goal state. If not, the loop is terminated and a graph of the nodes and their connections is displayed. The program successfully demonstrates the usefulness of these algorithms and how quickly they are able to reach a desired state.
