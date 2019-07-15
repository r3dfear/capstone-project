## Project description

The objective of this project was to write ROS nodes to implement core functionality of the autonomous vehicle system. This included traffic light detection, waypoint planner and control.

### Control strategy
The car behaviour is regulated by a FSM with two states: normal driving and braking.

The car drives normally when no traffic light is detected within a range of 100m or when the detected traffic light is green. This is obtained by setting the speed of all waypoints ahead to their default values.

As soon as a red traffic light is detected, the system computes the minimum braking distance to check if a braking is possible. If yes, a braking deceleration - dependent on the current speed and distance of the traffic light - is applied and the speed of all waypoints between the car and the traffic light is set so it gently decrease to zero at the light. All speeds of waypoints ahead of the traffic light are set to zero.

It's worth noting this scenario: a red light is detected and the car starts braking gently, if the light turns green while the car is braking, the car switches back to normal driving and accelerates to normal speed. This replicates the usual human behavior of not waiting the last moment to brake but to let go as soon as a red light is seen.

Throttle and brake pedal are then controlled by a PID, tuned with (parameters were taken from the last project and tuned afterwards):

| Parameter | Value  |
|-----------|--------|
| VEL_PID_P | 1.5    |
| VEL_PID_I | 0.005  |
| VEL_PID_D | 3      |

### Traffic Light Classification

I assume that there is only one traffic light color in Carlas field of view at the same time. Thus the detection with the highest score gets selected to identify the traffic light color. If there is no detection with a score higher than 50% probability, the classifier function returns "UNKNOWN".
The detection itself is done using a Convolutional Neural Network. I have used the TensorFlow Object Detection API and detection models, that are pre-trained on the COCO dataset. To be exact I have used "ssd_inception_v2_coco" model

#### Training
I have trainde only 1 classifier for simulator, however it would not be hard to train another one for real world, the only issue would be data(even then it is not that big of an issue), because I have ready code. The training was done using my personal PC and the following configuration parameters:

- num_classes: 4 ('Green','Red','Yellow','Unknown')
- num_steps: 10000
- max_detections_per_class: 10

Other parameters were used from the sample configuration files from tensorflow module (https://github.com/tensorflow/models/tree/master/research/object_detection/samples/configs).

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
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

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

### Other library/driver information
Outside of `requirements.txt`, here is information on other driver/library versions used in the simulator and Carla:

Specific to these libraries, the simulator grader and Carla use the following:

|        | Simulator | Carla  |
| :-----------: |:-------------:| :-----:|
| Nvidia driver | 384.130 | 384.130 |
| CUDA | 8.0.61 | 8.0.61 |
| cuDNN | 6.0.21 | 6.0.21 |
| TensorRT | N/A | N/A |
| OpenCV | 3.2.0-dev | 2.4.8 |
| OpenMP | N/A | N/A |

We are working on a fix to line up the OpenCV versions between the two.
