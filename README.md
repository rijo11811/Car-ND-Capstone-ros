# System Integration Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

This is the final project for the Udacity Self-Driving Car Engineer Nanodegree.  In this project, our team created several ROS nodes to implement core functionality of an autonomous vehicle.  For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

[//]: # (Image References)
[image1]: /imgs/carla_architecture.png
[image2]: /imgs/rosgraph.jpg
[image3]: /imgs/system_architecture.png
[image4]: /imgs/driv1.png
[image5]: /imgs/stop.jpg

![][image4]
![][image5]


## Setup

Please use **one** of the two installation options, either native **or** docker installation.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

## Project Overview

### Carla Architecture
Carla is the custom Lincoln MKZ that Udacity has converted into a self-driving car.  It's self-driving system is broken down into four major sub-systems: **Sensors**, **Perception**, **Planning** and **Control** 

![][image1]

#### Sensors
Sensor pack consists of **cameras**, **lidar**, **GPS**, **radar**, and **IMU**
#### Perception
Abstracts sensor inputs into object **detection** and **localization**
##### Detection
* Includes software pipelines for vehicle detection, traffic light detection, obstacle detection, etc
* Techniques in image manipulation include Histogram of Oriented Gradients (HOG) feature extraction, color transforms, spacial binning
* Methods of classification include sliding-window or sub-sampling along with heat maps and bounding boxes for recurring detections
##### Localization
* Answers the question: “Where is our car in a given map with an accuracy of 10cm or less?”
* Based on the notion that GPS is not accurate enough
* Onboard sensors are used to estimate transformation between measurements and a given map
#### Planning
Path planning is broken down into for sub-components: **route planning**, **prediction**, **behavioral planning**, and **trajectory planning**
##### Route Planning
The route planning component is responsible for high-level decisions about the path of the vehicle between two points on a map; for example which roads, highways, or freeways to take. This component is similar to the route planning feature found on many smartphones or modern car navigation systems.
##### Prediction
The prediction component estimates what actions other objects might take in the future. For example, if another vehicle were identified, the prediction component would estimate its future trajectory.
##### Behavioral Planning
The behavioral planning component determines what behavior the vehicle should exhibit at any point in time. For example stopping at a traffic light or intersection, changing lanes, accelerating, or making a left turn onto a new street are all maneuvers that may be issued by this component.
##### Trajectory Planning
Based on the desired immediate behavior, the trajectory planning component will determine which trajectory is best for executing this behavior.
### Control
The control component takes trajectory outputs and processes them with a controller algorithm like **PID** or **MPC** to adjust the control inputs for smooth operation of the vehicle. 

### ROS Architecture

The ROS Architecture consists of different nodes that communicate with each other through ROS messages. The nodes and their communication with each other are depicted in the picture below.

![][image2]

The styx_server links the simulator and ROS by providing information about the car's state and surroundings (car's current position, velocity and images of the front camera) and receiving control input (steering, braking, throttle). The other nodes can be associated with the three central tasks Perception, Planning and Control. 

In this project, I have used the traffic light state messages from the simulator to implement traffic light detectors in the car. A good addition to this would be the implication of a neural network which detects traffic lights and their states from the camera images. 

With the subscribed information of the traffic light detector and the subscriptions to base waypoints, the waypoint updater node is able to plan acceleration / deceleration and publish it to the waypoint follower node. This node publishes to the DBW (Drive by wire) which steers the car autonomously. It also takes as input the car's current velocity and outputs steering, braking and throttle commands. 

### Node Design

![][image3]

This section will discuss node design in detail

#### Waypoint Updater
This node performed following tasks:
Maintaining a list of waypoints ahead of the car: the car subscribes to a /base_)waypoints topic which is the list of complete waypoints the car has to traverse. It also subscribes to the position of the car. Using these values this node populates the list of waypoints ahead. In order to reduce latency this list has a size of 50. 

Updating the velocity of the waypoints ahead of the car:this node also receives the position of the traffic light stop line point ahead of the car. If this point lies in the path of the car, it progressively increases braking(Uniform deceleration of velocity) and comes to a complete stop at the stop line. 

#### Traffic Light Detection
The udacity capstone project simulator transmits the state of all the traffic lights. The tl_detector node subscribes to this message and the list of all the waypoints the car has to traverse and publishes the index of the waypoint which is closest to a red traffic light ahead of the vehicle.

#### Drive-By-Wire (DBW) Node
The third node written by us is the dbw_node which is responsible for steering the car. It subscribes to a twist controller which outputs throttle, brake and steering values with the help of a PID-controller and Lowpass filter. The dbw node directly publishes throttle, brake and steering commands for the car/simulator, in case dbw_enabled is set to true.


## Results

The car completed one lap inside the simulator without any issues.



