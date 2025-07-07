# Ubuntu 18.04 with NVIDIA driver support for CARLA DriveFuzz
FROM nvidia/cuda:12.1.0-devel-ubuntu18.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute

# Update package list and install essential packages
RUN apt-get update && apt-get install -y \
    software-properties-common \
    wget \
    curl \
    git \
    build-essential \
    cmake \
    ninja-build \
    libc6-dev \
    libstdc++-8-dev \
    clang-8 \
    clang++-8 \
    python3 \
    python3-pip \
    python3-dev \
    python3-setuptools \
    libpng-dev \
    libjpeg-dev \
    libtiff5-dev \
    libssl-dev \
    libffi-dev \
    libxml2-dev \
    libxslt1-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    unzip \
    rsync \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Set clang-8 as default compiler
RUN update-alternatives --install /usr/bin/clang clang /usr/bin/clang-8 100 \
    && update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-8 100

# Add workaround for libc++ atomic header issues
RUN mkdir -p /usr/local/include && \
    echo '#pragma GCC system_header' > /usr/local/include/atomic_fix.h && \
    echo '#include <atomic>' >> /usr/local/include/atomic_fix.h

# Install additional Python packages that might be needed
RUN python3 -m pip install --upgrade pip setuptools wheel

# Install common Python packages for autonomous driving simulation
RUN python3 -m pip install \
    numpy \
    opencv-python \
    matplotlib \
    scipy \
    Pillow \
    pygame \
    networkx \
    distro \
    pygame \
    docker \
    scikit-fuzzy \
    packaging

RUN python3 -m pip install -Iv setuptools==47.3.1

# Install Docker CE for Docker-in-Docker support
RUN apt-get update && \
    apt-get install -y \
        apt-transport-https \
        ca-certificates \
        gnupg \
        lsb-release \
        uidmap && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update && \
    apt-get install -y docker-ce docker-ce-cli containerd.io && \
    rm -rf /var/lib/apt/lists/*

# Install ROS Melodic
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && \
    curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - && \
    apt-get update && \
    apt-get install -y \
        ros-melodic-desktop-full \
        python-rosdep \
        python-rosinstall \
        python-rosinstall-generator \
        python-wstool && \
    rm -rf /var/lib/apt/lists/*

# Initialize rosdep
RUN rosdep init && rosdep update

# Set up ROS environment
RUN echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
ENV ROS_DISTRO=melodic
ENV ROS_PYTHON_VERSION=3

# Install additional ROS packages that might be useful for CARLA
RUN apt-get update && \
    apt-get install -y \
        ros-melodic-cv-bridge \
        ros-melodic-image-transport \
        ros-melodic-vision-opencv \
        ros-melodic-tf2-geometry-msgs \
        ros-melodic-tf2-sensor-msgs \
        ros-melodic-pcl-ros \
        ros-melodic-geographic-msgs \
        ros-melodic-carla-msgs \
        python-catkin-tools && \
    rm -rf /var/lib/apt/lists/*

# Create a working directory
WORKDIR /workspace

# Note: Project files will be mounted as a volume at runtime
# Use: docker run -v $(pwd):/workspace ... 

# Set up environment for CARLA building with CUDA 12.1 compatibility
ENV CC=/usr/bin/clang-8
ENV CXX=/usr/bin/clang++-8
ENV CXXFLAGS="-Wno-reserved-id-macro -Wno-deprecated-declarations -Wno-error=reserved-id-macro"
ENV CARLA_BUILD_NO_COLOR=true
ENV CFLAGS="-Wno-reserved-id-macro"


# Make sure the workspace directory is safe for git operations
RUN git config --global --add safe.directory /workspace

# Create Docker group and add to system (for Docker-in-Docker)
RUN groupadd docker || true

# Expose common ports (adjust as needed)
# CARLA ports: 2000-2002, ROS master: 11311
EXPOSE 2000 2001 2002 5000 11311

# Copy Docker entrypoint script for Docker-in-Docker support
RUN echo '#!/bin/bash\n\
# Create Docker daemon configuration for better compatibility\n\
mkdir -p /etc/docker\n\
cat > /etc/docker/daemon.json << EOF\n\
{\n\
  "storage-driver": "vfs",\n\
  "experimental": false,\n\
  "live-restore": false\n\
}\n\
EOF\n\
\n\
# Start Docker daemon if not running\n\
if [ ! -S /var/run/docker.sock ]; then\n\
    echo "Starting Docker daemon..."\n\
    dockerd --host=unix:///var/run/docker.sock --storage-driver=vfs &\n\
    sleep 10\n\
    echo "Docker daemon started"\n\
fi\n\
\n\
# Source ROS setup\n\
source /opt/ros/melodic/setup.bash\n\
\n\
# Execute the command\n\
exec "$@"' > /entrypoint.sh && \
    chmod +x /entrypoint.sh

# Set entrypoint
ENTRYPOINT ["/entrypoint.sh"]

# Default command
CMD ["/bin/bash"]

