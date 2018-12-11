# apriltag2_detection

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
