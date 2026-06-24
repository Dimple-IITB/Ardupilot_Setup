# Installing [Ardupilot](https://ardupilot.org/dev/docs/building-setup-linux.html) 
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git <br/>
cd ardupilot <br/>

Tools/environment_install/install-prereqs-ubuntu.sh -y <br/>
. ~/.profile <br/>

# Installing QGC
- [QGC install](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html)
- Use QGroundControl v5.0.8 - Stable
  ```https://github.com/mavlink/qgroundcontrol/releases/```

# Run Ardupilot and QGC together 
Terminal 1: 
```
cd ~/ardupilot/ArduCopter
sim_vehicle.py --console --map -w
````
Note: Use the above to check arducopter version as well <br/>
Terminal 2: 
```
./QGroundControl-x86_64.AppImage
```

Plan a path and make the drone move

# [ROS2](https://ardupilot.org/dev/docs/ros2-install.html#ros2-installation-ubuntu) with SITL
- Check ros distro in terminal `printenv ROS_DISTRO`

#### MAKING WORKSPACE
```
mkdir -p ~/ardu_ws/src
cd ~/ardu_ws
vcs import --recursive --input  https://raw.githubusercontent.com/ArduPilot/ardupilot/master/Tools/ros2/ros2.repos src

cd ~/ardu_ws
sudo apt update
rosdep update
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
````

##### Installing the Micro-XRCE-DDS-Gen build dependency:
```
sudo apt install default-jre
cd ~/ardu_ws
git clone --recurse-submodules --branch v4.7.0 https://github.com/ardupilot/Micro-XRCE-DDS-Gen.git
cd Micro-XRCE-DDS-Gen
./gradlew assemble
echo "export PATH=\$PATH:$PWD/scripts" >> ~/.bashrc
````

- Test Micro-XRCE-DDS-Gen installation
```
source ~/.bashrc
microxrceddsgen -help
# microxrceddsgen usage:
#     microxrceddsgen [options] <file> [<file> ...]
#     where the options are:
#             -help: shows this help
#             -version: shows the current version of eProsima Micro XRCE-DDS Gen.
#             -example: Generates an example.
#             -replace: replaces existing generated files.
#             -ppDisable: disables the preprocessor.
#             -ppPath: specifies the preprocessor path.
#             -I <path>: add directory to preprocessor include paths.
#             -d <path>: sets an output directory for generated files.
#             -t <temp dir>: sets a specific directory as a temporary directory.
#             -cs: IDL grammar apply case sensitive matching.
#     and the supported input files are:
#     * IDL files.
```

- BUILDING WORKSPACE
  ```
  cd ~/ardu_ws
  colcon build --packages-up-to ardupilot_dds_tests
  ```

##### ROS2 with SITL ()
- Ardupilot dependencies
  ```
  cd ardu_ws/src/ardupilot
  ./Tools/environment_install/install-prereqs-ubuntu.sh -y
  ```
- ROS2 packages ardupilot_msgs, micro_ros_agent, ardupilot_sitl and ardupilot_dds_tests as
  ```
  cd ardu_ws/
  colcon build --packages-up-to ardupilot_sitl
  ```
- source your workspace
  ```
  cd ~/ardu_ws
  source ./install/setup.bash
  ```
- Ardupilot 4.6
  ```
  source /opt/ros/humble/setup.bash
  cd ~/ardu_ws/
  colcon build --packages-up-to ardupilot_sitl
  source install/setup.bash
  ros2 launch ardupilot_sitl sitl_dds_udp.launch.py \
  transport:=udp4 \
  synthetic_clock:=True \
  wipe:=False \
  model:=quad \
  speedup:=1 \
  slave:=0 \
  instance:=0 \
  defaults:=$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/copter.parm,$(ros2 pkg prefix ardupilot_sitl)/share/ardupilot_sitl/config/default_params/dds_udp.parm \
  sim_address:=127.0.0.1 \
  master:=tcp:127.0.0.1:5760 \
  sitl:=127.0.0.1:5501
  ```
  - Once everything is running, you can now interact with ArduPilot through the ROS 2 CLI.
     ```
    source ~/ardu_ws/install/setup.bash
    # See the node appear in the ROS graph
    ros2 node list
    # See which topics are exposed by the node
    ros2 node info /ap
    # Echo a topic published from ArduPilot
    ros2 topic echo /ap/geopose/filtered
     ```
 - MAVProxy: To test and fly around, you can launch a mavproxy instance in yet another terminal:
    ```
    mavproxy.py --console --map --aircraft test --master=:14550
    ```
# ROS2, Ardupilot and Gazebo [Tutorial link](https://ardupilot.org/dev/docs/ros2-gazebo.html#ros2-gazebo)
- Installing [gazebo harmonic](https://gazebosim.org/docs/harmonic/install_ubuntu/)
  Check <br/>
  ```
  gz sim --version
  gz sim shapes.sdf
  ```
- Installing ros2 with gazebo
  ```
  cd ~/ardu_ws
  vcs import --input https://raw.githubusercontent.com/ArduPilot/ardupilot_gz/main/ros2_gz.repos --recursive src
  export GZ_VERSION=harmonic
  sudo apt install wget
  wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
  sudo apt update
  sudo wget https://raw.githubusercontent.com/osrf/osrf-rosdep/master/gz/00-gazebo.list -O /etc/ros/rosdep/sources.list.d/00-gazebo.list
  rosdep update
  cd ~/ardu_ws
  source /opt/ros/humble/setup.bash
  sudo apt update
  rosdep update
  rosdep install --from-paths src --ignore-src -y
  cd ~/ardu_ws
  colcon build --packages-up-to ardupilot_gz_bringup
  cd ~/ardu_ws
  source install/setup.bash
  colcon test --packages-select ardupilot_sitl ardupilot_dds_tests ardupilot_gazebo ardupilot_gz_applications ardupilot_gz_description ardupilot_gz_gazebo ardupilot_gz_bringup
  colcon test-result --all --verbose
  source install/setup.bash
  ros2 launch ardupilot_gz_bringup iris_runway.launch.py
  ```

# NOTES
LINKS: [ArduPilot](https://ardupilot.org/dev/docs/building-setup-linux.html), [ROS2](https://ardupilot.org/dev/docs/ros2-install.html#ros2-installation-ubuntu),[ROS2 with SITL](https://ardupilot.org/dev/docs/ros2-sitl.html)

IMP: [Ardu-ROS](https://ardupilot.org/dev/docs/ros2.html#ros2)



