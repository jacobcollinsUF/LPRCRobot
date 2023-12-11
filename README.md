# Cloning
- If you want to work on this robot in a home environment with the packages used in this project, run the command below.

        git clone --recurse-submodules https://github.com/jacobcollinsUF/LPRCRobot.git
- We are not sure the submodules were correctly uploaded when adding all of the files to the github. If this is the case and the command above does not work, you will have to manually git clone each package into your working source folder.
- For the pico code (LPRC_pico_uros), the lib file might have the same issue above where you would have to go and find the github that contains those libraries and git clone them in lib.
- *Warning*: even if cloning the files did work you may just have to create a new workspace from scratch anyways. A lot of these files have dependencies on other libraries/files that are on the computer of the robot. You can easily look up these packages in the Isaac ROS documentation and follow the tutorials to get everything set up.
# Ecosystem
- The robot currently runs Isaac ROS which is ROS2 humble in a docker with their specific architecture to take advantage of the cuda cores on the jetson devices.
- New packages can be created in either c++ or python.
- To build the code on the pico, your cmake arm compiler must be at least version 9.3.1
# LPRCRobot
- Senior Design project at UF designed to build a robot platform for future students to improve on.
- To get the robot moving, you first need to connect the ps4 controller. 
- To do so, open a terminal and enter the command:
  
        ds4drv --hidraw 
- Then hold down the home button and share button on the controller at the same time until the light starts blinking.
- Then go to settings->bluetooth and connect the controller.
- Once connected hit ctrl^C to stop hidraw.
- In the same terminal run the command below to build the Isaac ROS docker.
  

        cd ${ISAAC_ROS_WS}/src/isaac_ros_common && \
        ./scripts/run_dev.sh ${ISAAC_ROS_WS}
- Once the docker is built you need to rebuild the ydlidar SDK to set all of the files to the right path. Run the commands below:
        
        cd src/YDLidar-SDK/build
        sudo make install
        cd ../../..
- The ds4_driver package requires a couple of python packages to be installed in order to work. Run the two commands below:

        sudo apt install python3-evdev
        sudo apt install python3-pyudev
- Next run the command below to source and build the packages.

        cd /workspaces/isaac_ros-dev && \
        colcon build --symlink-install && \
        source install/setup.bash
- Next run the command below to connect the ps4 controller to the ROS2 ecosystem.

        ros2 launch ds4_driver ds4_driver.launch.xml use_standard_msgs:=true
- Open another terminal and run the same command above that was used to build the Isaac ROS docker.
- Then go ahead and run the command to source and build the packages
- Run the command below to set up twist messages from the controller.

        ros2 launch teleop_twist_joy teleop-launch.py joy_config:='xbox'
- Open another terminal and run the same command to build the Isaac ROS docker again.
- Then run the source and build command.
- Next run the two commands below to launch the micro ros bridge to get messages from the pico.

        sudo chmod 666 /dev/ttyACM0
        ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0
- DO NOT disconnect the pico or ps4 controller. Doing so will cause the docker to forget the devices exist. If you do disconnect the pico, you can run the sudo chmod command above to fix it. However, disconnecting the ps4 controller while in the docker may cause it to never remember it. If this happens you have to close all terminals and start from the beginning of all of these commands.
- To run the lidar run the two commands below:

        sudo chmod 666 /dev/ttyUSB0
        ros2 launch ydlidar_ros2_driver ydlidar_launch.py
- Right now the lidar only sweeps from 70 to -70 degrees. This was done to prevent the lidar from sensing the frame. We are not sure that it doesn't pick it up but future testing can easily be done to make sure.
# Pico Code
- All of the pico code was created by Jon Durrant. He has tutorials on how to use micro ros on the pico on his youtube and github page. This was very useful because he also is working on a robot that uses micro ros to drive the motors of the robot.
- The only changes we made to the code was to match the dimensions of our robot and the name of published nodes to match our robot. We also subscribe directly to /cmd_vel instead of robotname/cmd_vel.
- The pico currently publishes odometry messages and subscribes to twist messages. It does not use the HCSR04 library at all but we left those files in since you would have to go into each file and delete all of the necessary functions and commands. You would also have to append cmakelists.txt to ignore the path of the HCSR04 library if you wanted to do so.
- Jon Durrant github page:

    <https://github.com/jondurrant/DDD-Exp.git>
# Yolov8
- The yolov8 model on the robot is not currently a part of the ROS ecosystem but a new package could be made to do so or you could use the Isaac ROS package. However, using the Isaac ROS package will require a few workarounds to get working inside of the docker.
- To run the model, open a new terminal and run the commands below.

      cd yolov8_rs
      python3 yolov8_rs.py
- The model is connected to the realsense camera in the front and the model used can easily be changed by downloading a new one from the yolov8 github and editing yolov8_rs.py.
- This code was copied from the tutorial below:

    <https://www.youtube.com/watch?v=xqroBkpf3lY>
# Future Use
- We did not tune the realsense d435i imu parameters at all. So when using the realsense for VSLAM you may want to do so before hand.
- Installing the Nav2 ROS2 package can help you get started with autonomous movement of the robot.
- Twist muxes can be used to have the robot go from following a path to following people around.
- They are supposed to be coming out with a new pico in 2024 so it might be better to use that if it is faster. This can only work if all of the required SDK's and libraries work on the new pico. A pico W can be used to transmit the micro ros messages over UDP to the Orin Nano. This would free up one of the usba ports but transmission of messages may be slower.
- You can use the odometry messages produced by the pico but may not actually need them since the VSLAM node should produce odometry with the built-in IMU.
- Since the robot currently has to launch each package one at a time, it would be wise to create a package for the robot and then create a launch file that launches every node at once.
- We were unable to prevent having to "sudo make install" the YDLidar SDK each time we built the docker. There may be info out there on how to do so but we couldn't figure it out. You may have to do something similar to the vslam tutorial where they reference the realsense library in the docker.
- The python packages that you have to install each time can also be avoided by finding a workaround. However, this may not be needed if the ds4_driver package is no longer used.