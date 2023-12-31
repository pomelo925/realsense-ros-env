## ARM64 architecture ##
FROM ros:noetic 

LABEL org.opencontainers.image.authors="yoseph.huang@gmail.com"

ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=$USER_UID

### create non-root user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    ## passwordless
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

### pkgs installation
RUN apt-get update && apt-get install -y \
    sudo net-tools ssh nano git usbutils x11-apps \
    apt-utils ros-noetic-tf* \
    libssl-dev libusb-1.0-0-dev libudev-dev pkg-config udev \
    libx11-dev libxrandr-dev libxinerama-dev libxcursor-dev \
    libglu1-mesa-dev libglew-dev mesa-common-dev libxi-dev \
    ros-noetic-diagnostic-updater ros-noetic-rviz \
    && rm -rf /var/lib/apt/lists/*

### install realsense SDK
RUN git clone https://github.com/IntelRealSense/librealsense.git /opt/librealsense
WORKDIR /opt/librealsense/build
RUN cmake ../ -DBUILD_EXAMPLES=true -DCMAKE_BUILD_TYPE=Release && \
    make -j4 && make install

## Switch to non-root user 
USER $USERNAME
WORKDIR /home/$USERNAME

### ros pkg
WORKDIR /home/$USERNAME/realsense-ws/src
RUN git clone -b noetic-devel https://github.com/pal-robotics/aruco_ros.git && \
    git clone --branch 0.3.7 https://github.com/ros-drivers/usb_cam.git && \
    git clone https://github.com/ros-perception/image_pipeline.git && \
    git clone https://github.com/pal-robotics/ddynamic_reconfigure.git && \
    git clone --branch ros1-legacy --single-branch https://github.com/IntelRealSense/realsense-ros.git && \
    (cd realsense-ros && git checkout $(git tag | sort -V | grep -P "^2.\\d+\\.\\d+" | tail -1))

### install and build dependencies
RUN sudo apt-get update && \
    rosdep update && \
    rosdep install --from-paths . --ignore-src -r -y && \
    cd .. && \
    /bin/bash -c 'source /opt/ros/$ROS_DISTRO/setup.bash; catkin_make' && \
    echo "source /opt/ros/$ROS_DISTRO/setup.bash" >> ~/.bashrc \
    && sudo rm -rf /var/lib/apt/lists/* 

## Switch back to root user if necessary
# USER root

ENV SHELL /bin/bash

CMD ["/bin/bash"]
