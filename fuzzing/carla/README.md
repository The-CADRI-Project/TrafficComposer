# DriveFuzz Configuration for Non-"Ubuntu 18.04" Systems

Since Ubuntu 18.04 is officially EOL (End of Life), and DriveFuzz relies heavily on this specific system version, it will not work directly on other systems. This guide provides a solution using Docker-in-Docker techniques to create an Ubuntu 18.04 environment that works on any Docker-supported system.

- **System compatibility**: DriveFuzz requires Ubuntu 18.04, which is end of life.
- **Outdated resources**: Build resources for Carla 0.9.10.1 (default version for DriveFuzz) needed updates
- **Fixed issues**: Updated resource URLs and compiler arguments

## Prerequisites

- Any Linux computer with Docker properly installed and NVIDIA runtime supported 
- NVIDIA GPU (required - only NVIDIA GPUs are supported)
- Original DriveFuzz repository cloned

## Setup Instructions - Method 1

We have a forked DriveFuzz repo, in which we updated the corresponding resources and it is ready to use.

### Step 1. Build the Docker Environment

1. Copy the provided `Dockerfile` to the drivefuzz repo root directory, which we have included here as a git submodule. All the following commands regards the user in the current directory (`TrafficComposer/fuzzing/carla`)
```bash
cp Dockerfile ./drivefuzz/
```

2. Build the Docker image:

```bash
cd ./drivefuzz/ \
docker build -t drivefuzz-ubuntu1804 .
```

### Step 2. Start the Runtime Container

Run the following command to start the DriveFuzz container:

```bash
cd ./drivefuzz/ \
docker run -it -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/workspace --name drivefuzz_runner --network host drivefuzz-ubuntu1804
```

### 3. Build PythonAPI and Autoware Container
```bash
cd ./drivefuzz/carla \
make PythonAPI
```

## Setup Instructions - Method 2

### 1. Build the Docker Environment

1. Navigate to your cloned DriveFuzz repository directory
2. Copy the provided `Dockerfile` to the DriveFuzz root directory
3. Build the Docker image:

```bash
docker build -t drivefuzz-ubuntu1804 .
```

### 2. Start the Runtime Container

Run the following command to start the DriveFuzz container:

```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/workspace --name drivefuzz_runner --network host drivefuzz-ubuntu1804
```

**What this command does:**
- Maps the current directory (DriveFuzz repo + Dockerfile) to the container
- Enables Docker-in-Docker operations
- Uses host networking for optimal performance

### 3. Build PythonAPI and Autoware Container

1. **Replace Setup.sh file**: 
   - Override the original `Setup.sh` file in `drivefuzz/carla/Util/Buildtools` with the provided `Setup.sh`

2. **Build PythonAPI**:
   ```bash
   cd drivefuzz/carla
   make PythonAPI
   ```

## Notes

- This setup creates a containerized Ubuntu 18.04 environment
- All DriveFuzz operations should be performed within the container
- The container maintains access to Docker for nested operations

## Troubleshooting

**Traffic Manager Port Error**: If you encounter an error related to the traffic manager port during the fuzzing process, modify the port configuration:
1. Navigate to `drivefuzz/src/config.py`
2. Change `self.sim_tm_port` to `5000` instead of `8000`