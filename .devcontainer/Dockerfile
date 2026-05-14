FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV ROS_DISTRO=humble

RUN apt-get update && apt-get install -y \
    curl gnupg lsb-release software-properties-common \
    build-essential cmake git python3-pip \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
      -o /usr/share/keyrings/ros-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
       http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
       > /etc/apt/sources.list.d/ros2.list \
    && apt-get update \
    && apt-get install -y \
       ros-humble-desktop \
       python3-colcon-common-extensions \
       python3-vcstool \
       python3-rosdep \
       python3-colcon-mixin \
    && rm -rf /var/lib/apt/lists/*

RUN rosdep init 2>/dev/null || true && rosdep update

WORKDIR /ws_moveit
COPY src/ src/

RUN . /opt/ros/humble/setup.sh \
    && apt-get update \
    && rosdep install --from-paths src --ignore-src --rosdistro humble -y \
    && rm -rf /var/lib/apt/lists/*

RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc \
    && echo '[ -f /ws_moveit/install/setup.bash ] && source /ws_moveit/install/setup.bash' >> /root/.bashrc

CMD ["/bin/bash"]
