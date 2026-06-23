# Installing Ardupilot 
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git <br/>
cd ardupilot <br/>

Tools/environment_install/install-prereqs-ubuntu.sh -y <br/>
. ~/.profile <br/>

# ROS2 with SITL
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

##### ROS2 with SITL 
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
