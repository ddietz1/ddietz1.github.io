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
Designed and manufactured by BlueRobotics, the BlueROV2 
#### Newton Gripper
To effectively grip the bogies of the train for proper track alignment, our team created a set of custom grippers that could be mounted to the stock Franka end effector. The grippers were designed in Onshape and 3d printed in PLA. A thin foam layer was added as a final adjustment to allow firm grip without deforming the grippers.
#### USB Low Light Camera
The RealSense was selected for its ability to provide reasonably accurate depth information and its native compatibility with ROS2.

### Vision
I designed the vision stack as a two node ROS2 pipeline that turns the BlueROV2's UDP MJPEG stream into stable targets for easy use by the control node. The GStreamer pipeline recieved the ROV's video on a UDP port, converts frames to OpenCV BGR, re-encodes them as JPEG, and published them as a CompressedImage. Compressing the images is crucial to maintain a solid frame rate.

Once the compressed images are recieved by the object_detection node, they are decoded to BGR and segmented using HSV color thresholds. The node finds contours, selecting the largest, and computes the following: Normalized image-center errors for x and y, a normalized size metric used for determining distance given the lack of depth sensing from the camera, a circularity score based on the hull to ensure rejection of spurious shapes, a detected flag that is set to true if the same shape is detected for more than N frames. These metrics are published on a custom Object msg type to the control node.

<div style="text-align:center;">
  <img src="/assets/images/TrainingImages.webp" width="400"><br>
  <em>Training images used for the YOLO model.</em>
</div>

## System Flow

Once the launch file is running, the system stays in an IDLE state preventing cmd_vel messages from being published. To run the ROV, the ROV must be armed and manually set to SEARCHING mode using a custom service type in the bridge node. Once the system is searching, the ROV uses MagneticField messages from the onboard magnetometer to maintain its heading and executes a "lawnmover" style search with its onboard camera tilted at -40 degrees. Once an appropriate object is detected, the system moves to the RING_DETECTED state and publishes velocity messages to the bridge node to keep the object in the center of the frame and moving forward. Once the object is close enough, the control node adds a slight x offset to compensate for the offset between the gripper and the center of the ROV. When the object is within the error thresholds of the camera position and the size is large enough, the close gripper service is called to grab the object. The ROV then moves backward slightly while checking if the object is still detected. If it is not, it reenters searching mode and attempts the grab again. If the object is still detected, the system enters HOMING mode and slowly returns to the surface.

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
