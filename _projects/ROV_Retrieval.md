---
layout: project
title: "Autonomous Underwater Object Retrieval"

carousel_images:
  - /assets/images/FrankaArmProject.gif
carousel_height: 400px
carousel_width: auto
preview_gif: "/assets/images/450 Final Preview.gif"
code: "https://github.com/ME495-EmbeddedSystems/final-project-north-western-northwestern"

tags: ["ROS 2", "Python", "BlueROV2", "OpenCV", "YOLO", "MAVROS"]
date: 2025-11-15
description: "A system for identifying, navigating to, and retriving object underwater using the BlueROV2 platform."
---

## Overview

I am in the process of building a full ROS2 autonomy stack using Python to be used on the BlueROV2 by Blue Robotics. The overall core goal is autonomous searching, identification, navigation, and retrival of a diving ring from the bottom of the Northwestern pool. The system consists of two packages thus far for controlling the ROV. The system uses a custom ROS2 python API to convert ROS topic messages to Mavlink messages via Mavros bridge as well as nodes for retriving images from the onboard USB camera, a node for building robust object detection in OpenCV, and the controller node. This project is still in progress and is expected to be completed by March 13th, 2026.

## Subsystems

<div style="text-align:center;">
  <img src="/assets/images/Project-Setup-Annotated-2.webp" width="400"><br>
  <em>BlueROV2.</em>
</div>

### MAVROS Bridge
The BlueROV uses a Pixhawk autopilot running on Ardusub and thus cannot be directly communicated with using standard ROS2 topics. To allow for effective communication and control I implimented a node for converting Twist messages to MAVLINK messages. The bridge node publishes /mavros/command/send messages. When not being given velocity commands via teleop or the controller, the ROV is commanded to maintain a neutral position. The bridge node also controls other aspects of the ROV, such as the lights, Newton Undersea gripper, and USB camera pitch by publishing mavors/rc/override messages. The timer callbacks are implimented such that adding services for additional functionality is quite simple if you know the servo number that maps to that specific function.  

### Hardware

#### BlueROV2
The locomotive was a Piko DB class 191. Our test cars for railing were a Lionel NYC caboose and a 40ft reefer of unknown make. Our track was steel Bachmann E-Z track. The controller was a Kato analog controller.
#### Newton Gripper
To effectively grip the bogies of the train for proper track alignment, our team created a set of custom grippers that could be mounted to the stock Franka end effector. The grippers were designed in Onshape and 3d printed in PLA. A thin foam layer was added as a final adjustment to allow firm grip without deforming the grippers.
#### USB Low Light Camera
The RealSense was selected for its ability to provide reasonably accurate depth information and its native compatibility with ROS2.

### Vision
Our team designed the vision system to identify the trains, track, and any other aspects of the planning scene using a single Intel RealSense mounted to the end effector. 
To identify the various elements of the scene, a YOLO model was trained on sections of Bachmann track, two model trains, the model locomotive, and the control switch box. To gather the necessary training data, we took videos using the Franka-mounted RealSense. 

<div style="text-align:center;">
  <img src="/assets/images/TrainingImages.webp" width="400"><br>
  <em>Training images used for the YOLO model.</em>
</div>

We then used Grounding Dino and Meta’s SAM 2 to produce approximate bounding boxes from text descriptions of each car. Camera calibration was done with an Aruco marker to determine the distance/orientation of the camera relative to the table. The YOLO model outputs a Pytorch Tensor containing all data related to the minimum bounding boxes of the various elements in its immediate field of view. Our system extracts the corners of the bounding box and calculates the center pixel and orientation of the object. The system subscribes to two seperate vision topics to get the RGB and depth pixels of each object and uses the data from a seperate camera info topic to convert the pixel data to distances in meters in the planning scene.

## System Flow
Once the custom service is called, the Franka arm returns to its home pose. It then runs a custom scan function to identify all objects in the RealSense's field of vision. This function creates a dictionary of each detected object, runs until each object has 50 poses, then averages the results to ensure an accurate position, depth, and orientation relative to the camera. The robot proceeds to a position such that the locomotive is the only object in frame and runs this scan function again to achieve even more precise measurements of its position. This is done again for the track section on which the train cars will be placed. This core loop of move, scan, and average is performed again on the train cars in the staging area. The robot arm then selects each car in order and moves them to align with the track position. Once the car has been placed on the track, the grippers move to the front bogies to pinch and lift them slgihtly above the track. This is done again for the back bogies on each car. The reason for this is to align the bogies with the track position so the car will glide freely on the rails, this is necessary as the bogies will freely rotate while the train is being lifted by the robot and, if improperly aligned, the train will not be able to move along the track.

{% include video.liquid
  path="assets/images/TrainSuccess_fixed.mp4"
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
This project was planned and undertaken as an independent winter project as part of the Northwestern MSR program.
