# AGENTS.md

## Cursor Cloud specific instructions

### Product Overview

This is a **MoveIt 2** (v2.5.9) robotics motion planning workspace for **ROS 2 Humble**. It contains the full MoveIt 2 source stack organized as a standard colcon workspace under `src/`.

### Docker-based Development (Required)

The Cloud VM runs Ubuntu 24.04 Noble, but this codebase targets **ROS 2 Humble** (Ubuntu 22.04 Jammy). All building, testing, and running must be done inside the Docker container.

#### Build the Docker image (first time only)

```bash
sudo dockerd &>/tmp/dockerd.log &
sleep 3
cd /workspace
sudo docker build -f .devcontainer/Dockerfile -t moveit2-humble-dev .
```

#### Build the workspace from source

```bash
sudo docker run --rm \
  -v /workspace/src:/ws_moveit/src \
  -v /tmp/moveit_build:/ws_moveit/build \
  -v /tmp/moveit_install:/ws_moveit/install \
  -v /tmp/moveit_log:/ws_moveit/log \
  moveit2-humble-dev bash -c \
  "source /opt/ros/humble/setup.bash && cd /ws_moveit && \
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON \
   --event-handlers console_direct+"
```

#### Run tests

```bash
sudo docker run --rm \
  -v /workspace/src:/ws_moveit/src \
  -v /tmp/moveit_build:/ws_moveit/build \
  -v /tmp/moveit_install:/ws_moveit/install \
  -v /tmp/moveit_log:/ws_moveit/log \
  moveit2-humble-dev bash -c \
  "source /opt/ros/humble/setup.bash && source /ws_moveit/install/setup.bash && \
   cd /ws_moveit && colcon test && colcon test-result --all"
```

#### Launch MoveIt2 Panda demo

```bash
# Start container in background
sudo docker run -d --network host \
  -v /workspace/src:/ws_moveit/src \
  -v /tmp/moveit_build:/ws_moveit/build \
  -v /tmp/moveit_install:/ws_moveit/install \
  --name moveit-demo moveit2-humble-dev bash -c \
  "source /opt/ros/humble/setup.bash && source /ws_moveit/install/setup.bash && \
   ros2 launch moveit_resources_panda_moveit_config demo.launch.py use_rviz:=false"

# Check logs
sudo docker logs moveit-demo

# Execute commands inside running container
sudo docker exec moveit-demo bash -c \
  'source /opt/ros/humble/setup.bash && source /ws_moveit/install/setup.bash && ros2 topic list'

# Stop
sudo docker stop moveit-demo && sudo docker rm moveit-demo
```

### Key Gotchas

- **Docker required**: The dockerd daemon must be started before any Docker operations: `sudo dockerd &>/tmp/dockerd.log &`
- **Build volumes**: Build artifacts are stored in `/tmp/moveit_build` and `/tmp/moveit_install` on the host, mounted into the container. These persist across container runs but not across VM restarts.
- **Network mode**: Use `--network host` when running demos so ROS 2 DDS discovery works between containers and any host-side tools.
- **Source mounts**: The workspace source at `/workspace/src` is bind-mounted into the container at `/ws_moveit/src`, so code changes on the host are immediately reflected in the container.
- **56 packages**: The full workspace contains 56 colcon packages, all of which build successfully against ROS 2 Humble.
