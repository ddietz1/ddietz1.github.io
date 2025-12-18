---
layout: project
title: "Collision-free Object Manipulation"

carousel_images:
  - /assets/images/FrankaArmProject.gif
carousel_height: 400px
carousel_width: auto
preview_gif: "/assets/images/TrainRailing.gif"
code: "https://github.com/ME495-EmbeddedSystems/final-project-north-western-northwestern"

tags: ["ROS 2", "Python", "Franka Robot", "OpenCV"]
date: 2025-11-15
description: "A system for identifying and placing multiple model train cars onto a model track using a FR3 Franka Robot arm."
---

## Overview

This system utilizes a custom Python ROS2 API for the motion planning, robot state, and planning scene aspects of controlling the Franka robot arm. A custom package called franka_express uses OpenCV and YOLO for detection of the train cars, track, and track control switch. The package also contains a service call that runs the entire loop to create the planning scene, detect all relevant objects, pick and place each train car, and operate the control switch to run the train cars alongside a locomotive. The Franka arm was outfitted with custom grippers for use in fine maipulation of the train bogies, which require accuracy to within 1 millimeter in order to be properly placed onto the track. My primary contribution was contributing to the motion planning of the arm with ROS2, using OpenCV and YOLO bounding boxes to determine the positions of the objects and the appropriate motions to pick, place, and align the train cars and their bogies with the track.

