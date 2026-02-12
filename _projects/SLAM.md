---
layout: project
title: "Extended Kalman Filter SLAM from scratch"

carousel_images:
  - /assets/images/FrankaArmProject.gif
carousel_height: 400px
carousel_width: auto
preview_gif: "/assets/images/450 Final Preview.gif"
code: "https://github.com/ME495-Navigation/slam-ddietz1"

tags: ["ROS 2", "C++", "SLAM", "Turtlebot3"]
date: 2025-2-12
description: "EKF SLAM from scratch"
---

## Overview

I implimented an Extended Kalman filter (EKF) Simulataneous Localization and Mapping (SLAM) algorithm from scratch using several ROS2 packages and a custom C++ library. The algorithm is tested in the Rviz simulation environment and run on the Turtlebot3 robot. The turtlebot3 is equipped with a LIDAR scanner that provdes range and bearing measurements for obstacles in the environment. The EKF SLAM algorithm estimates the robot’s position and the position of the landmarks in the environment using these sensor measurements. The estimated robot position is compared to the ground truth position to evaluate the performance of the algorithm.

### SLAM Algorithm

### ROS2 Nodes

nuturtle_description: Contains the URDF description and visulization code for the turtlebot3 robot.

nusim: Contains the setup for the simulated environment for the turtlebot3. The packages contains launch files for starting the Rviz configuration for a single or multiple turtlebots as well as a launch file for running on an actual turtlebot. The package allows for the creation of stationary walls to mark the boundary of the simulation arena and as well as obstacles for the robot to avoid using markers. A service for teleporting the simulation robot back to its starting point is also provided.

nuturtle_control: The package provides operations responsible to control the turtlebot in real world and simulation. It includes three nodes: Circle, Turtle_control, and Odometry. Circle is responsible to make the robot move in a Circle at a set velocity and radius. Turtle_control is responsible of converting twist messages to wheel commands. The Odometry node is responsible for providing odometry calculations.

turtlelib: While not a ROS2 node, turtlelib is a 2D geometry library containing structures for twists, transforms, vectors, and points. Turtlelib provides classes and functions for performing kinematics calculations on the turtlebot.

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

## Background
The system was developed as part of an individual final project for Northwestern University's Mechanical Engineering 495: Sensing, Navigation, and Machine Learning for Robotics.
