# MoveIt 2 工作区使用文档

这份文档面向第一次接触本项目的学习者，帮助你快速理解这个仓库是什么、从哪里开始看、如何构建运行，以及后续怎样做二次开发。

## 1. 项目一句话概览

本仓库是一个已经拉取并构建过的 ROS 2 `colcon` 工作区，核心内容围绕 MoveIt 2 运动规划框架展开。它包含 MoveIt 2 本体、官方教程、机器人描述资源、MoveIt Task Constructor、RViz 可视化辅助工具，以及若干底层工具库。

MoveIt 2 的典型用途包括：

- 给机械臂做运动规划、碰撞检测和轨迹执行。
- 在 RViz 中交互式配置起点、目标点和规划场景。
- 使用 C++ 或 Python 编写机器人运动控制程序。
- 使用 MoveIt Task Constructor 组合抓取、放置等复杂任务流程。

## 2. 你需要先了解的背景知识

如果你刚开始学习，建议按下面顺序补齐基础：

1. **Linux 与终端基础**：能使用 `cd`、`source`、环境变量、包管理器。
2. **ROS 2 基础**：理解 node、topic、service、action、parameter、launch file。
3. **colcon 工作区**：理解 `src/`、`build/`、`install/`、`log/` 的作用。
4. **机器人描述文件**：大致知道 URDF 描述机器人结构，SRDF 描述 MoveIt 语义信息。
5. **MoveIt 基础概念**：planning scene、planning group、kinematics、collision checking、trajectory。

如果这些概念还不熟，可以先完成 ROS 2 官方教程中创建 package、编写 publisher/subscriber、使用 launch file 的部分，再回到本项目。

## 3. 根目录结构

```text
/workspace
├── src/       # 源码区：所有 ROS 2 包和外部仓库都在这里
├── build/     # colcon 构建中间产物，通常不要手动修改
├── install/   # colcon 安装结果，运行前需要 source 这里的 setup.bash
├── log/       # colcon 构建与测试日志
└── README.md  # 本文档
```

重点只需要关注 `src/`。`build/`、`install/`、`log/` 是构建生成目录，出问题时可以查看日志，但日常开发不要直接改里面的文件。

## 4. `src/` 下主要模块

| 路径 | 作用 | 建议阅读顺序 |
| --- | --- | --- |
| `src/moveit2` | MoveIt 2 主仓库，包含核心库、ROS 接口、规划器、RViz 插件、Python 接口等 | 先看整体，再按需求深入 |
| `src/moveit2_tutorials` | MoveIt 2 官方教程源码，也是学习入口 | 第一个看 |
| `src/moveit_resources` | 测试和教程使用的机器人资源，包括 Panda、Fanuc、PR2 等 | 跑 demo 时配合看 |
| `src/moveit_task_constructor` | 用于组合多阶段任务的 MoveIt Task Constructor 框架 | 掌握基础规划后再看 |
| `src/moveit_visual_tools` | 在 RViz 中显示 marker、轨迹、碰撞物体等调试信息 | 做可视化调试时看 |
| `src/srdfdom` | SRDF 解析与写入库 | 研究机器人语义配置时看 |
| `src/launch_param_builder` | 在 ROS 2 launch 文件中组织参数的 Python 工具 | 写 launch 文件时看 |
| `src/rosparam_shortcuts` | 参数读取辅助库 | 维护旧接口或相关依赖时看 |

## 5. MoveIt 2 内部怎么分层

`src/moveit2` 是最重要的目录，可以先按下面的结构理解：

| 路径 | 说明 |
| --- | --- |
| `src/moveit2/moveit_core` | 核心算法库，包括运动学模型、碰撞检测接口、规划插件接口、控制器和传感器接口等 |
| `src/moveit2/moveit_ros` | ROS 2 集成层，把 core、规划器、RViz、move_group 等功能组装起来 |
| `src/moveit2/moveit_planners` | 规划器相关实现，例如 OMPL、CHOMP、Pilz 等 |
| `src/moveit2/moveit_kinematics` | 运动学求解器插件 |
| `src/moveit2/moveit_plugins` | 控制器接口等插件 |
| `src/moveit2/moveit_py` | MoveIt 2 的 Python 接口 |
| `src/moveit2/moveit_setup_assistant` | MoveIt Setup Assistant，用于生成机器人 MoveIt 配置 |
| `src/moveit2/moveit_runtime` | 运行时聚合包 |

学习时不要一开始就逐行读 `moveit_core`。更有效的方式是先运行教程 demo，再从一个实际流程反查代码，例如：

1. `ros2 launch moveit2_tutorials demo.launch.py`
2. RViz 中点击 `Plan`
3. 追踪 `move_group`、planning interface、planner plugin、robot model 的调用链

## 6. 环境准备

本工作区面向 ROS 2 Humble 生态，`moveit2_tutorials.repos` 中的主要仓库也指向 `humble` 或 ROS 2 对应分支。

如果你的终端还没有设置 ROS 发行版变量，先执行：

```bash
export ROS_DISTRO=humble
```

### 6.1 基础依赖

在一台新机器上，需要先安装：

```bash
source /opt/ros/$ROS_DISTRO/setup.bash
sudo apt install python3-rosdep python3-colcon-common-extensions python3-colcon-mixin python3-vcstool
sudo rosdep init
rosdep update
colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml
colcon mixin update default
```

如果已经初始化过 `rosdep`，再次运行 `sudo rosdep init` 可能会提示已存在，这是正常情况。

### 6.2 安装系统依赖

在工作区根目录执行：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
sudo apt update
rosdep install -r --from-paths src --ignore-src --rosdistro $ROS_DISTRO -y
```

### 6.3 可选：切换 DDS 实现

官方教程建议使用 Cyclone DDS：

```bash
sudo apt install ros-$ROS_DISTRO-rmw-cyclonedds-cpp
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

如果你把这行写进 `~/.bashrc`，后续所有新终端都会使用 Cyclone DDS。

## 7. 构建项目

### 7.1 完整构建

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
colcon build --mixin release
```

机器内存较小时可以降低并行度：

```bash
colcon build --mixin release --parallel-workers 1
```

### 7.2 Debug 构建

学习和调试 C++ 代码时可以使用：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
colcon build --mixin debug
```

### 7.3 构建单个包

只改了某个包时，优先构建该包：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
colcon build --packages-select srdfdom
```

如果要同时构建依赖它的包：

```bash
colcon build --packages-up-to srdfdom
```

## 8. 每次运行前必须 source 环境

构建完成后，打开新终端需要执行：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
```

如果忘记 source，常见现象包括：

- `ros2 launch` 找不到 package。
- `ros2 run` 找不到可执行文件。
- Python import 找不到 MoveIt 相关模块。
- 新构建的 package 没有出现在 ROS 2 环境中。

## 9. 常用运行命令

### 9.1 运行 MoveIt 2 RViz 快速 demo

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 launch moveit2_tutorials demo.launch.py
```

这个命令会启动 Panda 机械臂示例、MoveIt 相关节点和 RViz。它是学习本项目最重要的入口。

如果想从空 RViz 配置开始，手动添加 Motion Planning 插件：

```bash
ros2 launch moveit2_tutorials demo.launch.py rviz_config:=panda_moveit_config_demo_empty.rviz
```

进入 RViz 后，可以尝试：

1. 添加或选择 `MotionPlanning` 插件。
2. 设置 planning group 为 `panda_arm`。
3. 拖动目标位姿 marker。
4. 点击 `Plan` 生成轨迹。
5. 点击 `Plan & Execute` 在仿真中执行轨迹。

### 9.2 运行 MoveIt Visual Tools demo

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 launch moveit_visual_tools demo_rviz.launch.py
```

这个 demo 适合学习如何在 RViz 中显示轨迹、碰撞物体、marker 和机器人状态。

### 9.3 运行 MoveIt Task Constructor demo

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 launch moveit_task_constructor_demo demo.launch.py
```

这个 demo 展示一个基于 Panda 的简单 pick and place 流程。建议在理解基本 MoveIt 规划流程后再看。

### 9.4 运行 SRDF 解析示例

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 run srdfdom display_srdf src/srdfdom/test/resources/pr2_desc.3.srdf
```

这个命令可以帮助你理解 SRDF 文件中 group、end effector、disable collision 等语义配置如何被解析。

## 10. 推荐学习路线

### 阶段 1：先跑起来

目标：知道项目能做什么，看到机械臂在 RViz 中规划运动。

阅读和操作：

1. 阅读 `src/moveit2_tutorials/doc/tutorials/getting_started/getting_started.rst`。
2. 执行一次完整构建。
3. 运行 `ros2 launch moveit2_tutorials demo.launch.py`。
4. 在 RViz 中完成一次 `Plan`。

完成标准：

- 能解释 `source install/setup.bash` 的作用。
- 能解释 RViz 中 planning group、start state、goal state 是什么。
- 能手动触发一次 Panda 机械臂规划。

### 阶段 2：理解 MoveIt 的基本使用方式

目标：从“会点按钮”进阶到“能写一个简单程序调用 MoveIt”。

阅读和操作：

1. 阅读 `src/moveit2_tutorials/doc/tutorials/quickstart_in_rviz/quickstart_in_rviz_tutorial.rst`。
2. 阅读 `src/moveit2_tutorials/doc/tutorials/your_first_project/your_first_project.rst`。
3. 创建一个依赖 `moveit_ros_planning_interface` 和 `rclcpp` 的 C++ package。
4. 使用 `MoveGroupInterface` 设置目标位姿、规划并执行。

完成标准：

- 知道 `move_group` 是 MoveIt 的核心运行节点之一。
- 知道 `MoveGroupInterface` 是用户程序调用 MoveIt 的常用 C++ 接口。
- 知道为什么运行自己的程序前需要先启动 demo launch。

### 阶段 3：看懂机器人配置

目标：理解一个机器人是如何接入 MoveIt 的。

重点目录：

- `src/moveit_resources/panda_description`
- `src/moveit_resources/panda_moveit_config`
- `src/moveit_resources/fanuc_description`
- `src/moveit_resources/fanuc_moveit_config`

建议重点看：

- URDF/Xacro：机器人连杆、关节、惯性、几何模型。
- SRDF：planning group、虚拟关节、end effector、禁用碰撞对。
- `kinematics.yaml`：运动学求解器配置。
- `joint_limits.yaml`：速度、加速度限制。
- `ompl_planning.yaml`：OMPL 规划器配置。
- `moveit_controllers.yaml`：控制器配置。

完成标准：

- 能说清 `description` 包和 `moveit_config` 包分别负责什么。
- 能找到 Panda 的 planning group 配置。
- 能解释一个机器人接入 MoveIt 大致需要哪些配置文件。

### 阶段 4：深入 MoveIt 内部

目标：开始读 MoveIt 2 源码。

推荐顺序：

1. `src/moveit2/moveit_ros/planning_interface`：用户侧接口。
2. `src/moveit2/moveit_ros/move_group`：MoveIt 运行时核心节点。
3. `src/moveit2/moveit_core/robot_model`：机器人模型。
4. `src/moveit2/moveit_core/planning_scene`：规划场景与碰撞环境。
5. `src/moveit2/moveit_planners/ompl`：OMPL 规划器集成。
6. `src/moveit2/moveit_ros/visualization`：RViz 插件与可视化。

建议带着问题读源码：

- RViz 点 `Plan` 后，请求是怎么到 `move_group` 的？
- `move_group` 如何选择 planning pipeline？
- 碰撞检测在什么时候发生？
- 规划结果如何被转换成轨迹并显示？
- `MoveGroupInterface` 和 ROS action/service/topic 的关系是什么？

### 阶段 5：学习复杂任务规划

目标：理解 pick and place 这类多阶段任务如何组合。

重点目录：

- `src/moveit_task_constructor/core`
- `src/moveit_task_constructor/demo`
- `src/moveit_task_constructor/visualization`
- `src/moveit2_tutorials/doc/tutorials/pick_and_place_with_moveit_task_constructor`

学习重点：

- stage：单个任务阶段。
- container：顺序或并行组织多个 stage。
- planning scene：阶段之间传递解和环境状态。
- introspection：查看每个阶段的成功、失败和候选解。

## 11. 本地构建教程网站

如果想把 `moveit2_tutorials` 生成 HTML 文档：

```bash
cd /workspace/src/moveit2_tutorials
source /opt/ros/$ROS_DISTRO/setup.bash
./build_locally.sh
```

如果依赖已经安装过，可以跳过安装步骤：

```bash
./build_locally.sh noinstall
```

如果想边改边自动重建：

```bash
./build_locally.sh noinstall loop
```

生成结果在：

```text
src/moveit2_tutorials/build/html/index.html
```

## 12. 日常开发流程

### 12.1 修改代码前

```bash
cd /workspace
git status
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
```

确认工作区干净，并确认当前终端已经加载 ROS 2 和本工作区环境。

### 12.2 修改后构建

优先构建你改动的包：

```bash
colcon build --packages-select <package_name>
```

如果不确定依赖关系，构建到该包为止：

```bash
colcon build --packages-up-to <package_name>
```

### 12.3 运行测试

运行单包测试：

```bash
colcon test --packages-select <package_name> --event-handlers console_direct+
colcon test-result --verbose
```

示例：

```bash
colcon test --packages-select srdfdom --event-handlers console_direct+
colcon test-result --verbose
```

### 12.4 查看 package

```bash
ros2 pkg list | grep moveit
ros2 pkg prefix moveit2_tutorials
ros2 pkg executables srdfdom
```

## 13. 常见问题排查

### 13.1 找不到 package

现象：

```text
Package 'moveit2_tutorials' not found
```

处理：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 pkg prefix moveit2_tutorials
```

如果仍然找不到，重新构建对应包。

### 13.2 构建时缺依赖

处理：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
rosdep install -r --from-paths src --ignore-src --rosdistro $ROS_DISTRO -y
```

### 13.3 RViz 没有显示机器人

检查：

- 是否启动了正确的 launch 文件。
- `Fixed Frame` 是否设置正确，例如 Panda demo 中常见为 `panda_link0`。
- Motion Planning 插件的 `Robot Description` 是否为 `robot_description`。
- 终端里是否有 robot description、planning scene、TF 相关错误。

### 13.4 自己写的 MoveIt 程序启动后等待或报错

如果出现类似找不到 `robot_description` 的错误，通常是因为没有先启动包含 `move_group` 和机器人描述的 launch 文件。

先启动：

```bash
ros2 launch moveit2_tutorials demo.launch.py
```

再在另一个终端运行你的程序。

### 13.5 改了代码但运行行为没变

检查：

```bash
cd /workspace
colcon build --packages-select <package_name>
source install/setup.bash
```

如果是 Python 或 launch 文件，也要确认使用的是当前工作区的 package：

```bash
ros2 pkg prefix <package_name>
```

## 14. 推荐从哪里开始看代码

如果你的目标是学习使用 MoveIt：

1. `src/moveit2_tutorials/doc/tutorials/quickstart_in_rviz`
2. `src/moveit2_tutorials/doc/tutorials/your_first_project`
3. `src/moveit_resources/panda_moveit_config`
4. `src/moveit2/moveit_ros/planning_interface`

如果你的目标是理解 MoveIt 内部：

1. `src/moveit2/moveit_core/README.md`
2. `src/moveit2/moveit_ros/README.md`
3. `src/moveit2/moveit_ros/move_group`
4. `src/moveit2/moveit_core/planning_scene`
5. `src/moveit2/moveit_planners/ompl`

如果你的目标是做抓取、放置、多阶段任务：

1. `src/moveit_task_constructor/README.md`
2. `src/moveit_task_constructor/demo`
3. `src/moveit2_tutorials/doc/tutorials/pick_and_place_with_moveit_task_constructor`
4. `src/moveit_task_constructor/core`

如果你的目标是用 Python：

1. `src/moveit2/moveit_py/README.md`
2. `src/moveit2/moveit_py/moveit`
3. MoveIt 2 tutorials 中与 Python 或 Jupyter 相关的章节

## 15. 术语速查

| 术语 | 含义 |
| --- | --- |
| `colcon` | ROS 2 常用构建工具 |
| `workspace` | ROS 2 工作区，通常包含 `src/`、`build/`、`install/`、`log/` |
| `package.xml` | ROS package 元信息和依赖声明 |
| `launch file` | 启动多个 ROS 节点和参数的入口 |
| `URDF` | 机器人几何、关节、连杆等物理描述 |
| `SRDF` | MoveIt 使用的语义描述，例如规划组和末端执行器 |
| `planning scene` | MoveIt 中的世界模型，包括机器人状态和碰撞环境 |
| `planning group` | 一组用于规划的关节，例如 `panda_arm` |
| `move_group` | MoveIt 规划和执行能力的核心运行节点 |
| `MoveGroupInterface` | C++ 用户程序常用的 MoveIt 调用接口 |
| `RViz` | ROS 可视化工具 |
| `MTC` | MoveIt Task Constructor，用于复杂任务级规划 |

## 16. 最小实践路径

如果你想用最短路径上手，可以只做下面这些：

```bash
cd /workspace
source /opt/ros/$ROS_DISTRO/setup.bash
source install/setup.bash
ros2 launch moveit2_tutorials demo.launch.py
```

然后在 RViz 中：

1. 找到 Motion Planning 面板。
2. 选择 `panda_arm`。
3. 拖动目标机械臂姿态。
4. 点击 `Plan`。
5. 观察生成的轨迹。

做完这一步后，再阅读：

```text
src/moveit2_tutorials/doc/tutorials/your_first_project/your_first_project.rst
```

它会带你写第一个 C++ MoveIt 程序，这是从“使用 demo”进入“自己开发”的最佳起点。
