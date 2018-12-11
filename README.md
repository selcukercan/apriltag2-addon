# apriltag2_detection

For the April Tag detection pipeline we require two different container: the base container and the apriltah-addon container.

To work with the tested versions:

Launch the base container by executing

```shell
docker -H ![ROBOT_NAME].local run -it --net host --privileged -v /data:/data --memory="300m" --memory-swap="2.0g" --name base selcukercan/frpi-base:v5 /bin/bash
```

While in the __base container__ start the camera, decoding and rectification nodes by launching

```shell
roslaunch pi_camera camera_apriltag_demo.launch veh:=![ROBOT_NAME]
```

Then in a new terminal window, launch the apriltag-addon container by executing

```shell
docker -H ![ROBOT_NAME].local run -it --net host --privileged -v /data:/data --memory="300m" --memory-swap="2.0g" --name base selcukercan/apriltag2add-on:v11 /bin/bash
```

Now start the apriltag core algorithm by launching.

```shell
roslaunch apriltags2_ros apriltag2_demo.launch veh:=mete
```

At this point you should be able to see the results produced by the apriltag algoritm by inspecting the output of

```shell
rostopic echo /![ROBOT_NAME]/tag_detections
```

Note that this returns the pose of the camera frame with respect to the aprtiltag frame. Details on coordinate frame conventions can be found under [here](https://github.com/selcukercan/apriltag2_detection/blob/master/src/apriltags2_ros/apriltags2_ros/include/apriltags2_ros_post_process/rotation_utils.py).  

To get the position in fixed frame XYZ Euler Angles

```shell
roslaunch apriltags2_ros detection_to_local_frame.launch veh:=![ROBOT_NAME]
```

To inspect the orientation expressed in Euler Angles topic that ends with ´tag_detections_local_frame´
