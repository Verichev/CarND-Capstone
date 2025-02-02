# Self Driving Car - Capstone Project
<img src="./docs/imgs/kitt.gif" width="1000">

## Table of Contents

- [Overview](#overview)
	- [Team Members](#team-members)
- [Architecture](#architecture)
- [Components](#components)
	- [Sensory Subsystem](#sensory-subsystem)
	- [Perception Subsystem](#perception-subsystem)
	- [Planning Subsystem](#planning-subsystem)
	- [Control Subsystem](#control-subsystem)
- [Results](#results)
	- [Simulation Mode](#simulation-mode)
	- [Site Mode](#site-mode)
- [Limitations & Future Enhancements](#limitations--future-enhancements)


## Overview

### Team Members

By order of joining the team:

- Ali Kureishy (Lead) (safdar.kureishy@gmail.com)
- Eugene Verichev (verichev.iyc@gmail.com)
- Szilard Bessenyei (bessszilard@gmail.com)
- Naveed Usmani (naveedhd@gmail.com)
- Mark Melnykowycz (mark.melnykowycz@gmail.com)

## Architecture

This project aims to integrate different components of autonomous driving onto a ROS platform that can be installed on a specific vehicle (a Lincoln HKZ sedan) and would, in the absence of any obstacles, achieve single-lane navigation that obeys traffic lights.

ROS provides a platform and framework, with a vast library of hardware integrations, for building software-controlled robotics systems, including self-driving cars, drones, personal home robots etc. Though it is not the only platform available for such purposes, it has wide research adoption and is gaining commercial use as well.

The platform operates as a collection of processes ('nodes'), that utilize the ROS platform for message-based event-driven asynchronous communication, through an abstraction called a 'topic', to which these nodes can attach themselves as subscribers (observers) and publishers (observables).

Here is an architectural illustration of the components:
![Architecture](docs/imgs/architecture.png)

## Components

There are 4 high-level subsystems generally found in a self-driving system:

- Sensors: How the vehicle senses information about its surroundings
- Perception: How the vehicle attaches meaning/semantics/understanding to the sensory information it receives
- Planning: How the vehicle reacts to the perceived semantics (the brains of the car)
- Control: Actuates the decisions from the planner (such as with steering, throttle, brake etc)

Below we discuss our implementation, as it relates to the components above.

### Sensory Subsystem

### Perception Subsystem

#### Traffic light detector/classifier:

##### Topics Involved

![TLDetectorTopics](docs/imgs/tl_detector_topics.png)

##### Data Set

The dataset was downloaded from [here](dataset_link). Beside that dataset, we labeled images manually with labelImg.
![labelImg](docs/imgs/labeling.png)

We had three classes: 1 - Green, 2 - Yellow, 3 - Red.

##### Training the model

We chose the transfer learning technique to solve traffic light classification. We fine-tuned the ssd_mobilenet_v2 model from the Tensorflow model zoo. We made the following significant changes:

1. We decreased the last fully connected layer from 90 to 3 nodes.
2. Increased the box predictor size from 1 to 3.
3. We enabled depth wise convolution.
4. We reduced the training process steps from 200k to 20k.
5. We changed the paths for tune checkpoint, input, and label map path.

The model was trained on Google Cloud ML and locally as well with the following configuration:

|Batch Size |Steps |Learning Rate |Anchors Min Scale |Anchors Max Scale |Anchors Aspect Ratio |
|---        |---   |---           |---               |---               |---                  |
|24         |20000 |0.004         |0.1               |0.5               |0.33                 |

The used scripts for traning are located in [utils folder]

##### Model Evalation:

![Simulation results](docs/imgs/combine_sim.jpg)
*Results for Udacity sumlation*

![Training bag results](docs/imgs/combine_valid.jpg)
*Results for training bag*


### Planning Subsystem

This is where the autonomy is implemented. Though there are numerous components that would fall in this category, the scope of this document will be limited only to the components implemented in this particular project. These components are as discussed below.

#### Behavioral Planning -- Waypoint Updater

Waypoint updater provides the waypoints, starting from the car to some points ahead, that the car will follow. The updater will replan at every time step to update the waypoint but also gracefully decelerate the
car incase of traffic light is red or there are obstacles ahead.

This node receives map information, localization (car's current location in the map) and traffic light status (by traffic waypoint) and decelerrates the car to a stop if the traffic light is red.

waypoint_updater node subscribes to following:

- `/base_waypoints` (styx_msgs/Lane)
- `/current_pose` (geometry_msgs/PoseStamped)
- `/traffic_waypoint` (std_msgs/Int32)

and publishes the following topics:

- `/final_waypoints` (styx_msgs/Lane)

![WaypointUpdaterTopics](docs/imgs/waypoint_updater_topics.png)

The deceleration relation w.r.t distance from the lane end is roughly following:
![distance-vs-velocity](docs/imgs/dist-velocity.png)

#### Path Planning -- Waypoint Follower

No changes were made to this component for this project. However, at a high leve, this piece implements Path Planning by determining the trajectory that the car needs to take, split into time-steps, which then dictates the controls that would get sent to the control subsystem for steering and throttle.

### Control Subsystem

#### Twist Controller

#### Drive-By-Wire Interface (DBW)

![DBWTopics](docs/imgs/dbw_topics.png)


## Results

### Simulation Mode

### Site Mode

## Limitations & Future Enhancements

### Obstacle detection/avoidance

### Lane change

### Pedestrian tracking

### Other vehicle tracking
