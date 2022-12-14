# social_nav_sim
A ros package for GAZEBO simulation of socially-aware robot navigation methods such as DSRNN(https://github.com/Shuijing725/CrowdNav_DSRNN.git), ORCA, Soical Force, etc.

A number of walking or standing people and various types of static objects are simulated in GAZEBO with random or manually set initial positions and goals.

# Prerequisite
This package has been verified in ubuntu 20.04, ROS Noetic, GAZEBO 11, Python3.8, PyTorch 1.7.1 with CUDA 11.0 and cuDNN 8.0.5.
<details>
<summary>NVIDIA Driver, CUDA Toolkit, and cuDNN</summary>

Refer to the links below.

- **NVIDIA Driver**(apt installation recommended): https://phoenixnap.com/kb/install-nvidia-drivers-ubuntu

- **CUDA Toolkit**:
  - Installation: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation
  - Environment setup after installation: https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#environment-setup

- **cuDNN**: https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html#cudnn-package-manager-installation-overview

</details>

<details>
<summary>ROS Noetic(supported in Ubuntu 20.04)</summary>

Install ROS Noetic(desktop-full) following the instructions in http://wiki.ros.org/noetic/Installation/Ubuntu.

</details>

<details>
<summary>Python Packages</summary>

Enter the following command in a terminal.
```
pip3 install numpy lxml
```

</details>

<details>
<summary>PyTorch</summary>

Install PyTorch 1.7.1 using the following command referenced from https://pytorch.org/get-started/previous-versions/
```
pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 -f https://download.pytorch.org/whl/torch_stable.html
```

</details>

<details>
<summary>Python-RVO2</summary>

Refer to the following link: https://github.com/sybrenstuvel/Python-RVO2

</details>

<details>
<summary>ROS packages</summary>

```
suo apt install ros-noetic-velodyne-driver ros-noetic-velodyne-description ros-noetic-velodyne-gazebo-plugins ros-noetic-velodyne-msgs
```

</details>

# Installation
Clone this repository in your catkin workspace and build.
```
cd $(your catkin workspace)/src
git clone https://github.com/drancon/social_nav_sim.git
catkin build
source $(your catkin workspace)/devel/setup.bash
```

# Example
1. Launch one of files in `main` package.
```
roslaunch main 10s10d_orca.launch
```
2. Wait unitl all modules are ready.

<img src="/readme_figures/ready.png" width="1080" />

3. Call a ros service named `/environment/power_switch` to start a simulation episode.
- The simulation can be turned on and off multiple times by calling the service repeatedly
```
rosservice call /environment/power_switch "{}"
```

<img src="/readme_figures/start.png" width="1080" />

4. Watch the simulation running.

<img src="/readme_figures/simulation.png" width="1080" />

5. A message announcing the terminal state of episode and statistics of all episodes will be displayed in the terminal

<img src="/readme_figures/termination.png" width="1080" />

6. Either one of following actions can be done:
- Terminate the simulation by entering `ctrl + c` in the terminal.
- Reset the environment for another simulation episode by calling a ros service named `/environment/reset`.
```
rosservice call /environment/reset "{}"
```

# How to run your own checkpoint file
1. Put your checkpoint file in `agent/param/ckpts` folder.
```
cp $(checkpoint file name).pt $(your catkin workspace)/src/social_nav_sim/agent/param/ckpts/$(checkpoint file name).pt
```
2. create a new folder to contain parameters for your model by copying an existing folder in `main/param`
```
cd $(your catkin workspace)/src/social_nav_sim/main/param/
cp -r 5d_abs_holo/ $(new model name)/
```
3. Change values in parameter files in the newly created folder to fit your checkpoint file's environment setting.

**Noticeable Parameters**
<details open>
<summary>environment.yaml</summary>

- auto_start (whether to automatically start the simulation when agent and obstacles are ready): `True` or `False`
- auto_termination (whether to automatically finish the simulation when the agent reaches the terminal state): `True` or `False`
  
</details>

<details open>
<summary>agent.yaml</summary>

- robot
  - kinematics (kinematics of robot): `"holonomic"` or `"unicycle"`
  - policy (policy of robot): `"srnn"`, `"orca"`, or `"social_force"`
- policy
  - srnn
    - device (pytorch computing device): `"cuda"` or `"cpu"`
    - coord_frame (reference coordinate frame of srnn): `"absolute"` or `"relative"`
    - ckpt_file (name of your checkpoint file): `$(checkpoint file name).pt`
    - robot_state_size (number of variables given to dsrnn as robot_state): `7`(absolute) or `4`(relative)
  
</details>

<details open>
<summary>obstacle.yaml</summary>

- num_obstacle (total number of obstacles): `any positive integer`
- num_static (number of static obstacles): `any positive integer' or '0`
- initial_position_list (manually set initial position of obstacles): `[[x1,y1], [x2,y2], ...]` or `[]`(all random initial position)
- object_name_list (names of objects to be spawned as static obstacles in the environment): `["oak_tree", "trash_bin", ...]` or `[]`(no objects)
  
</details>

4. Copy a launch file in `main/launch` to create a new launch file and change value of argument `configuration` in the newly created launch file to your paremeter folder.
```
cd $(your catkin workspace)/src/social_nav_sim/main/launch/
cp 5d_abs_holo.launch $(new model name).launch
```

# References
- agent & simulation: DSRNN(https://github.com/Shuijing725/CrowdNav_DSRNN.git)
- robot model: Humic(https://github.com/SeoulTech-HCIRLab/Humic.git)
- animated_obstacle_plugin: modification of gym_ped_sim/actor_plugin(https://github.com/onlytailei/gym_ped_sim.git)
