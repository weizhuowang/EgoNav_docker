# Dockerfile for Smartbelt development
# Based on Ubuntu 20.04 + ROS Noetic + PyTorch 1.13

FROM --platform=linux/amd64 tiryoh/ros-desktop-vnc:noetic

RUN echo "root:1" | chpasswd

# Set environment variables for user creation
ARG USERNAME=ubuntu
ARG PASSWORD=1

# Create user and set password with sudo privileges
RUN useradd -m -s /bin/bash -G adm,sudo $USERNAME && \
    echo "$USERNAME:$PASSWORD" | chpasswd && \
    echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Set the default user
# USER $USERNAME

# Personal perference
RUN cd /home/ubuntu/ && mkdir Documents && mkdir Downloads
RUN chown ubuntu:ubuntu /home/ubuntu/Documents
RUN chown ubuntu:ubuntu /home/ubuntu/Downloads
# Install Opencv 3.4.16
    # Get ready
RUN apt-get update && apt-get upgrade -y
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget unzip software-properties-common
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential cmake git pkg-config libgtk-3-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
    gfortran openexr libatlas-base-dev python3-dev python3-numpy \
    libtbb2 libtbb-dev libdc1394-22-dev

    # Download
RUN cd /home/ubuntu/ && mkdir -p /home/ubuntu/opencv/build
RUN cd /home/ubuntu/opencv && \
    wget https://github.com/opencv/opencv/archive/3.4.16.zip && \
    unzip 3.4.16.zip && \
    rm 3.4.16.zip && \
    wget https://github.com/opencv/opencv_contrib/archive/3.4.16.zip && \
    unzip 3.4.16.zip && \
    rm 3.4.16.zip

    # Build
RUN cd /home/ubuntu/opencv/build && \
    cmake -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-3.4.16/modules ../opencv-3.4.16/ && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local  -D INSTALL_C_EXAMPLES=ON  -D INSTALL_PYTHON_EXAMPLES=ON  -D OPENCV_GENERATE_PKGCONFIG=ON  -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib-3.4.16/modules -D BUILD_EXAMPLES=ON ../opencv-3.4.16 && \
    make -j10


# Install RealSense SDK
RUN mkdir -p /etc/apt/keyrings
RUN curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | tee /etc/apt/keyrings/librealsense.pgp > /dev/null
RUN apt-get install apt-transport-https
RUN echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
    tee /etc/apt/sources.list.d/librealsense.list
RUN apt-get update && apt-get install -y librealsense2-utils=2.53.1* librealsense2-gl=2.53.1* librealsense2-net=2.53.1* librealsense2-dkms librealsense2=2.53.1* librealsense2-dev=2.53.1* librealsense2-dbg=2.53.1*



# TODO: problem here
# RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE && \
#     add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u && \
#     apt-get update && apt-get install -y --no-install-recommends librealsense2-dkms librealsense2-utils librealsense2-dev

# RUN mkdir -p /etc/apt/keyrings && \
#     curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | tee /etc/apt/keyrings/librealsense.pgp > /dev/null
# RUN apt-get install apt-transport-https
# RUN echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
#     tee /etc/apt/sources.list.d/librealsense.list
# RUN apt-get update && apt-get install -y librealsense2-dkms librealsense2-utils librealsense2-dev

# Install Eigen 3.4rc-1 (3.3.91)
RUN cd /home/ubuntu/Downloads && wget https://gitlab.com/libeigen/eigen/-/archive/3.4-rc1/eigen-3.4-rc1.zip && \
    unzip eigen-3.4-rc1.zip && \
    rm eigen-3.4-rc1.zip && \
    mv /usr/include/eigen3 /usr/include/eigen3bu && \
    mkdir /usr/include/eigen3 && \
    mv /home/ubuntu/Downloads/eigen-3.4-rc1/Eigen /usr/include/eigen3/Eigen && \
    mv /home/ubuntu/Downloads/eigen-3.4-rc1/unsupported /usr/include/eigen3/unsupported && \
    mv /home/ubuntu/Downloads/eigen-3.4-rc1/signature_of_eigen3_matrix_library /usr/include/eigen3/signature_of_eigen3_matrix_library && \
    rm -rf /home/ubuntu/Downloads/eigen-3.4-rc1

# Install conda and (ros_conda) env
RUN cd /home/ubuntu/Downloads && \
    wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" -O /home/ubuntu/Downloads/miniforge3.sh && \
    bash /home/ubuntu/Downloads/miniforge3.sh -b -p /home/ubuntu/miniforge3 && \
    rm /home/ubuntu/Downloads/miniforge3.sh && \
    eval "$(/home/ubuntu/miniforge3/bin/conda shell.bash hook)" && \
    conda init bash

# Install any ROS package that is still missing
RUN apt-get install -y ros-noetic-pcl-conversions ros-noetic-pcl-ros ros-noetic-ddynamic-reconfigure

USER ubuntu

ENV PATH=/home/ubuntu/miniforge3/bin:${PATH}
SHELL ["/bin/bash", "--login", "-c"]

# Install ros_conda env and packages in a single layer
RUN . /home/ubuntu/miniforge3/etc/profile.d/conda.sh && \
    . /home/ubuntu/miniforge3/etc/profile.d/mamba.sh && \
    mamba init bash && \
    mamba create -n ros_conda python=3.8 -y && \
    mamba activate ros_conda && \
    mamba install -y \
        numpy \
        scipy \
        matplotlib \
        scikit-learn \
        pytorch \
        torchvision \
        torchaudio \
        pytorch-cuda=11.8 \
        -c pytorch -c nvidia && \
    mamba install -y \
        pytorch-lightning \
        -c conda-forge && \
    mamba clean -afy

RUN . /home/ubuntu/miniforge3/etc/profile.d/conda.sh && \
    mamba activate ros_conda && \
    pip install --no-cache-dir \
        pyyaml \
        rospkg \
        tensorboard \
        torchsummary \
        pycryptodomex \
        gnupg \
        opencv-python \
        joblib

RUN . /home/ubuntu/miniforge3/etc/profile.d/conda.sh && \
    mamba activate ros_conda && \
    pip install --no-cache-dir \
        empy==3.3.4

RUN cd /home/ubuntu/Documents && \
    git clone https://<yourghptoken>@github.com/armlabstanford/ARMLab_smart_belt && \
    cd ARMLab_smart_belt/catkin_ws
    # source /opt/ros/noetic/setup.bash && \
    # . /home/ubuntu/miniforge3/etc/profile.d/conda.sh && \
    # conda activate ros_conda && \
    # catkin_make && \
    # source devel/setup.bash

COPY ./entrypoint.sh /
USER root
ENTRYPOINT [ "/bin/bash", "-c", "/entrypoint.sh" ]