# AGENTS.md

## Cursor Cloud specific instructions

### Product Overview

This is a **MoveIt 2** robotics motion planning workspace for ROS 2. It contains the full MoveIt 2 source stack (moveit2, moveit2_tutorials, moveit_task_constructor, moveit_visual_tools, moveit_resources, etc.) organized as a standard colcon workspace under `src/`.

### Important: ROS 2 Distro Mismatch

The source code in `src/` is written for **ROS 2 Humble** (MoveIt 2 v2.5.9), but the Cloud VM runs **Ubuntu 24.04 Noble** which only supports **ROS 2 Jazzy**. Key consequences:

- **C++ tutorial code** (e.g., `src/moveit2_tutorials/`) will not compile against Jazzy due to API changes (`trajectory_` → `trajectory`, `error_code_` → `error_code`, `computeCartesianPath` signature changes).
- **MoveIt 2 core** from source also has minor Jazzy incompatibilities (missing `ament_target_dependencies` for `pluginlib` in some sub-packages).
- **Binary Jazzy packages** (`ros-jazzy-moveit*`) are installed and fully functional. Use these for running demos and testing.
- Resource/config packages (`moveit_resources_*_description`, `srdfdom`, `moveit_visual_tools`, `rosparam_shortcuts`, `launch_param_builder`, `moveit_configs_utils`) **do build** from source against Jazzy.

### Compiler Gotcha

The VM's default `c++` is symlinked to `clang++`, which cannot find `libstdc++`. The update script reconfigures `c++` → `g++` via `update-alternatives`. If you see linker errors about `-lstdc++`, verify `c++ --version` shows `g++`, not `clang++`.

### Running MoveIt 2

```bash
source /opt/ros/jazzy/setup.bash

# Launch Panda demo (headless, no RViz)
ros2 launch moveit_resources_panda_moveit_config demo.launch.py use_rviz:=false

# Launch with RViz (requires display)
ros2 launch moveit_resources_panda_moveit_config demo.launch.py
```

### Building from source

```bash
source /opt/ros/jazzy/setup.bash
cd /workspace

# Build only the compatible packages (resources, configs, utilities)
colcon build --packages-select \
  launch_param_builder moveit_common moveit_configs_utils \
  srdfdom moveit_visual_tools rosparam_shortcuts \
  --cmake-args -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_COMPILER=/usr/bin/g++ -DCMAKE_C_COMPILER=/usr/bin/gcc
```

### Running tests

```bash
source /opt/ros/jazzy/setup.bash
source /workspace/install/setup.bash   # if workspace was built
colcon test --packages-select <package_name>
colcon test-result --all
```

### Lint

```bash
source /opt/ros/jazzy/setup.bash
ament_lint_cmake <CMakeLists.txt>
ament_copyright <package_dir>/
ament_xmllint <package.xml>
```
