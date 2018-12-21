# apriltag2_detection
This package is used to analyze the relative pose of single duckiebot estimated by using AprilTag2. Except for the normal pipeline of AprilTag2, it also provides post-processing srcipts to get result analysis, as well as several unit-tests for testing the correctness of the system.

## result analysis
This package provide `detection_post_process.launch` to analyze the performance of relative pose estimation by using AprilTag2. Each time it collects a certain number of estimated pose (both postion and rotation) and then compute the following:
* max, min, range, mean, variance of these pose estimation.
* time used for each subprocess in the pipeline of relative pose estimation (detecting AprilTag + computing relative pose).
* cpu/ram comsumption for relative pose estimation.

click to open the generated document which is located at `![APRILTAG_ADDON_ROOT]/fapriltag2_addon/src/apriltags2_ros/apriltags2_ros/docs/build/html`
=======
1. `catkin_make`
2. `source devel/setup.bash`
3. `roslaunch pi_camera camera_apriltag_demo.launch veh:=duckiebot`
4. `roslaunch apriltags2_ros apriltag2_demo.launch veh:=duckiebot`
5. `roslaunch apriltags2_ros detection_to_local_frame.launch veh:=duckiebot`
6. `rosparam set enable_statistics true` & `rosrun rosprofiler rosprofiler`
7. `roslaunch apriltags2_ros detection_post_process.launch veh:=duckiebot`

## unit test
Several test are provided in this package for testing the correctness of the system.

### detection_to_local_frame_testor_node.test
1. This is used to test the correctness of pose estimation. All images in _tests/test_image_ are tested and the name of the images are their groundtruth in degree. If you want to specify one image, for now you should put only that image in the above folder.
2. In the test, each time we publish one of these images as well as camera info. `apriltag_detector_node` is launched to detect apriltag and publish pose estimation (camera frame with respect to tag frame).  `detection_to_local_frame` is launched for coverting pose estimation into appropriate frame (robot wrt world), which is the ultimate estimation of pose we are using for tests.


```shell
cd [REPO_ROOT]
catkin_make
```

```shell
source act_addon_environment.sh
```

```shell
rostest apriltags2_ros detection_to_local_frame_testor_node.test
```

* `rostest --text apriltags2_ros detection_to_local_frame_testor_node.test` to get console output to the screen

## Duckietowm-Spesific

April Tag detection pipeline uses two container: the base container and the apriltag-addon container. This documemt prepend each command with the name of the environment in which that command needs to be run, i.e. ´[LOCAL_PC]´ represents that the command should be run on your local PC. Make sure to remove the environment identifier ´[ENVIRONMENT_NAME]´ before executing the command.

First set the `DOCKER_HOST` on your terminal

```shell
[LOCAL_PC] export DOCKER_HOST=![ROBOT_NAME].local
```

Then, launch the base container by executing

```shell
[LOCAL_PC] docker -H ![ROBOT_NAME].local run -it --net host --privileged -v /data:/data --memory="300m" --memory-swap="2.0g" --name base selcukercan/frpi-base:v5 /bin/bash
```

While in the __base container__ roslaunch the file that starts the camera, decoding and rectification nodes

```shell
[BASE_CONTAINER] roslaunch pi_camera camera_apriltag_demo.launch veh:=![ROBOT_NAME]
```

Then open up a new container and set ´DOCKER_HOST´ in this window as well.
```shell
[LOCAL_PC] export DOCKER_HOST=![ROBOT_NAME].local
```

In the new terminal window, launch the __apriltag-addon__ container

```shell
[LOCAL_PC] docker -H ![ROBOT_NAME].local run -it --net host --privileged -v /data:/data --memory="300m" --memory-swap="2.0g" --name base selcukercan/apriltag2add-on:v12 /bin/bash
```

Now you will be inside `selcukercan/apriltag2add-on`. Next, start the apriltag core algorithm

```shell
[APRILTAG_ADDON_CONTAINER] roslaunch apriltags2_ros apriltag2_demo.launch veh:=mete
```

At this point you should be able to see the results produced by the apriltag algorithm by inspecting

```shell
[APRILTAG_ADDON_CONTAINER] rostopic echo /![ROBOT_NAME]/tag_detections
```

Note that this returns the pose of the camera frame with respect to the apriltag frame. To learn more about the coordinate frame definitions/conventions, please refer to [this](https://github.com/selcukercan/apriltag2_detection/blob/master/src/apriltags2_ros/apriltags2_ros/include/apriltags2_ros_post_process/rotation_utils.py).  

To get the position in fixed frame XYZ Euler Angles

```shell
[APRILTAG_ADDON_CONTAINER]  roslaunch apriltags2_ros detection_to_local_frame.launch veh:=![ROBOT_NAME]
```

To inspect the orientation expressed in Euler Angles

```shell
[APRILTAG_ADDON_CONTAINER] rostopic echo ![ROBOT_NAME]/apriltags2_ros/publish_detections_in_local_frame/tag_detections_local_frame
```

## Documentation

This project uses sphinx to generate project documentation automatically.

To start generating documents you have to first install sphinx

```shell
sudo apt-get install python-sphinx
```
generate the documentation

```shell
cd ![APRILTAG_ADDON_ROOT]/src/apriltags2_ros/apriltags2_ros/docs
make html
```
