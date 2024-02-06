# 4.4 Car Body Tracking

## Introduction

The Car Body Tracking app is used to control the robot to follow the movement of the human body. The app consists of MIPI image acquisition, body detection and tracking, body tracking strategy, image coding and decoding, and a web display interface. The workflow is shown in the following diagram:

![](./image/car_tracking/body_tracking_workflow.jpg)

The app is demonstrated using a virtual car in the PC-side Gazebo simulation environment, but the control commands can also be directly used to control a physical car.

Code Repository: <https://github.com/HorizonRDK/body_tracking>

## Supported Platforms

| Platform | Operating System | Example Functionality                  |
| ---------| ---------------- | ------------------------------------- |
| RDK X3, RDK X3 Module, RDK Ultra | Ubuntu 20.04  | Start MIPI/USB camera to capture images, perform body keypoints detection and body tracking, and display the tracking effect in Gazebo |

## Preparation

### Horizon RDK Platform

1. Horizon RDK has been flashed with the Ubuntu 20.04 system image provided by Horizon.

2. TogetheROS.Bot has been successfully installed on Horizon RDK.

3. MIPI or USB camera has been installed on Horizon RDK.

4. The PC used for Horizon RDK should be in the same network segment (either wired or connected to the same wireless network, with the first three parts of the IP address being consistent). The PC should have the following environment installed:

   - Ubuntu 20.04 system
   - [ROS2 Foxy Desktop](https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html)
   - Gazebo and Turtlebot3 related packages can be installed using the following commands:

    ```shell
    sudo apt-get install ros-foxy-gazebo-*
    sudo apt install ros-foxy-turtlebot3
    sudo apt install ros-foxy-turtlebot3-simulations
    ```

## User Guide

### Horizon RDK Platform

After running the Car Body Tracking app, the car motion control package will select the human body closest to the front of the car (with the largest width of the body detection box) as the tracking target. When the human body is far from the car, the car starts to move forward to approach the body and keeps it in front of the car.After the APP is launched, the sensor will publish images and corresponding algorithm results, which can be rendered and displayed on the PC browser. (Enter http://IP:8000 in the browser, where IP is the IP address of the Horizon RDK).

Launch the simulation environment on the PC side:

```shell
source /opt/ros/foxy/setup.bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo empty_world.launch.py
```

After successful launch, the car effect in the simulation environment is as follows:

![](./image/car_gesture_control/gazebo.jpeg)


**Publish images using MIPI camera**

```shell
# Configure the tros.b environment
source /opt/tros/setup.bash

# Copy the configuration file needed for running the example from the installation path of TogetheROS.
cp -r /opt/tros/lib/mono2d_body_detection/config/ .

# Configure the MIPI camera
export CAM_TYPE=mipi

# Launch the launch file
ros2 launch body_tracking body_tracking_without_gesture.launch.py
```

**Publish images using USB camera**

```shell
# Configure the tros.b environment
source /opt/tros/setup.bash

# Copy the configuration file needed for running the example from the installation path of tros.b.
cp -r /opt/tros/lib/mono2d_body_detection/config/ .

# Configure the USB camera
export CAM_TYPE=usb

# Launch the launch file
ros2 launch body_tracking body_tracking_without_gesture.launch.py
```

## Result Analysis

The following information is outputted in the terminal when running on the Horizon RDK board.[body_tracking-7] [WARN] [1653430533.523069034] [ParametersClass]: TrackCfg param are
[body_tracking-7] activate_wakeup_gesture: 0
[body_tracking-7] track_serial_lost_num_thr: 100
[body_tracking-7] activate_robot_rotate_thr: 45
[body_tracking-7] activate_robot_move_thr: 5
[body_tracking-7] move_step: 0.3
[body_tracking-7] rotate_step: 0.5
[body_tracking-7] img_width: 960
[body_tracking-7] img_height: 544
[body_tracking-7] 
[body_tracking-7] [WARN] [1653430533.712812076] [TrackingManager]: update frame_ts 395787, 873
[body_tracking-7] [WARN] [1653430533.713105576] [TrackingManager]: Tracking body start!, track_id: 1, frame_ts: 395787, tracking_sta(0:INITING, 1:TRACKING, 2:LOST): 1, gesture: 0
[body_tracking-7] [WARN] [1653430535.018442618] [TrackingManager]: Do move! body_rect_width: 353, thr: 864, move_step_ratio: 1, body_rect_to_top: 20, img_height: 544, move_step: 0.3
[body_tracking-7] [WARN] [1653430535.220268535] [TrackingManager]: Do rotate move, ts sec: 3397, nanosec: 387800000
[body_tracking-7] [WARN] [1653430535.220408576] [RobotCmdVelNode]: RobotCtl, angular: 0 0 0, linear: 0.3 0 0, pub twist ts: 1653430535220394

以上log截取了一段App启动后的输出。启动后先打印相关配置（TrackCfg param）。检测到人体后小车就开始进入跟随状态（tracking_sta值为1），并以0.3m/s的速度前进运动（RobotCtl, angular: 0 0 0, linear: 0.3 0 0）靠近人体。

PC端在终端使用`ros2 topic list`命令可以查询到地平线RDK的topic信息：

```shell
$ ros2 topic list
/camera_info
/cmd_vel
/hbmem_img04054242060426080500012020112713
/hobot_mono2d_body_detection
/image
/parameter_events
/rosout
```
Among them, `/image` is the image captured by the Horizon RDK from the MIPI sensor and encoded in JPEG format, `/hobot_mono2d_body_detection` is the algorithm message published by the Horizon RDK which contains the human body detection results, and `/cmd_vel` is the motion control command published by the Horizon RDK.

PC端在终端使用`ros2 topic echo /cmd_vel`命令可以查看到地平线RDK发布的运动控制指令：

```shell
linear:
  x: 0.5
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: -0.5
---
linear:x: 0.5
y: 0.0
z: 0.0
angular:
x: 0.0
y: 0.0
z: -0.5
---
```

In the PC simulation environment, the car follows the movement of the human body. The simulated car movement effect is as follows:

![](./image/car_tracking/tracking.gif)