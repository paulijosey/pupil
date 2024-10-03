FROM ros:humble-perception
ARG USERNAME=samrtglasses
ARG USER_UID=1000
ARG USER_GID=1000
ARG WS=/pupil_dev/pupil
ARG ROS_WS=/pupil_dev/ros_ws

ENV PIP_BREAK_SYSTEM_PACKAGES 1

# Create the user
RUN groupadd --gid $USER_GID $USERNAME
RUN useradd --uid $USER_UID --gid $USER_GID -g video -m $USERNAME
# [Optional] Add sudo support. Omit if you don't need to install software after connecting.
RUN apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*


# [Optional] Uncomment this section to install additional OS packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y \
    curl \
    git \
    gnupg \
    lsb-release \
    sudo \
    unzip \
    wget \
    zip \
    udev \
    cmake \
    pkg-config \
    mesa-utils \
    libglu1-mesa-dev \
    freeglut3-dev \
    mesa-common-dev \
    libglew-dev \
    libglfw3-dev \
    libglm-dev \
    libao-dev \
    libmpg123-dev \
    python3-pip \
    python3-dev \
    libxkbcommon-dev \
    libxinerama-dev \
    libxcursor-dev \
    libxi-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install GLFW
RUN mkdir -p /tmp/glfw \
    && cd /tmp/glfw \
    && git clone https://github.com/glfw/glfw.git \
    && cd glfw \ 
    && cmake . \
    && make -j4 \
    && make install 

# Copy environment.yml (if found) to a temp location so we update the environment. Also
# copy "noop.txt" so the COPY instruction does not fail if no environment.yml exists.
COPY ../requirements.txt .devcontainer/noop.txt /tmp/python-requirements/
RUN if [ -f "/tmp/python-requirements/requirements.txt" ]; then \
    pip install --no-cache-dir -r /tmp/python-requirements/requirements.txt ; fi

RUN mkdir -p ${WS}

RUN . /opt/ros/${ROS_DISTRO}/setup.sh \
    && mkdir -p ${ROS_WS}/src \
    && cd ${ROS_WS}/src \
    && cd ${ROS_WS} \
    && colcon build

# install custom ros messages/packages
RUN cd ${ROS_WS}/src/ \
    && git clone https://github.com/paulijosey/pupil_labs_ros2.git
RUN . /opt/ros/${ROS_DISTRO}/setup.sh \
    && cd ${ROS_WS} \
    && colcon build


RUN chown -R $USERNAME:$USERNAME ${WS}

# fix wierd ROS docker memory issue
RUN echo "ulimit -n 524388" >> /home/${USERNAME}/.bashrc
RUN echo "ulimit -Hn 524388" >> /home/${USERNAME}/.bashrc
RUN echo "ulimit -n 524388" >> /root/.bashrc
RUN echo "ulimit -Hn 524388" >> /root/.bashrc

# add ros setup file to bashrc for convinience
RUN echo source /opt/ros/${ROS_DISTRO}/setup.bash >> /home/${USERNAME}/.bashrc
RUN echo source /opt/ros/${ROS_DISTRO}/setup.bash >> /root/.bashrc
# add new ros setup to bashrc if it exists
RUN echo "[ -f ${ROS_WS}/install/setup.bash ] && source ${ROS_WS}/install/setup.bash" >> /home/${USERNAME}/.bashrc
RUN echo "[ -f ${ROS_WS}/install/setup.bash ] && source ${ROS_WS}/install/setup.bash" >> /root/.bashrc

USER ${USERNAME}
CMD ["/bin/bash"]
ENV SHELL /bin/bash
SHELL ["/bin/bash", "-c"]


