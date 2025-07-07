# DevContainer and GitHub Actions Overview

## DevContainer Configuration

### Main Configuration Files

#### devcontainer.json
- **Purpose**: Main devcontainer configuration using a pre-built Docker image
- **Image**: `johnny555/bar_ws_25:latest`
- **User**: `ubuntu`
- **Workspace**: Mounted to `/workspace`
- **Port Forwarding**: 6080:80 for VNC access
- **Environment**: Sets `DISPLAY=:1` for GUI applications
- **VS Code Extensions**: Includes ROS, Python, C++, and robotics development tools
- **Settings**: Configures ROS Jazzy environment and code formatting

#### devcontainer-docker_compose.json
- **Purpose**: Alternative devcontainer configuration using Docker Compose
- **Compose File**: Points to `docker-compose-wsl.yml`
- **Service**: Uses `bar-desktop-full` service
- **Usage**: Rename to `devcontainer.json` to use Docker Compose approach instead of single image

### Docker Compose Files

#### docker-compose.yml (Default)
- **Image**: `johnny555/bar:v4`
- **User**: `ubuntu`
- **Volume**: Mounts `../` to `/workspace:cached`
- **Environment**: `DISPLAY=:1`
- **Port**: 6080:80
- **Command**: Keeps container running with sleep loop

#### docker-compose-ubuntu.yml & docker-compose-wsl.yml
- Platform-specific variations of the main compose file
- Optimized for Ubuntu and WSL environments respectively

### Dockerfiles

#### Dockerfile
- **Purpose**: Simple image modification for workspace compatibility
- **Base Image**: `johnny555/scr:v2`
- **Modification**: Updates path references from `/home/ubuntu/start_creating_robots2` to `/workspace` in bashrc
- **Usage**: Adapts existing image for devcontainer workspace mounting

#### Dockerfile_scrv2
- **Purpose**: Complete ROS2 desktop environment image definition
- **Base Image**: `ubuntu:noble-20240605`
- **Components**:
  - Ubuntu MATE desktop environment
  - VNC server (TigerVNC) + noVNC web interface for browser access
  - ROS2 Jazzy desktop installation with development tools
  - Development applications (VSCodium, Firefox, build tools)
  - Extensive robotics packages (Gazebo, MoveIt, Navigation2, SLAM, etc.)
  - Custom setup via `setup_container.bash` during build
- **Output**: Creates the base image published as `johnny555/bar_ws_25:latest`
- **Relationship**: This Dockerfile builds the image that docker-compose files consume

**Note**: Dockerfiles are **input files** used to build custom Docker images, not outputs from docker-compose. However, these Dockerfiles have **minimal direct impact** on the current devcontainer setup since:

- **devcontainer.json** pulls pre-built `johnny555/bar_ws_25:latest` from Docker Hub
- **docker-compose.yml** uses pre-built `johnny555/bar:v4` from Docker Hub
- Neither configuration builds images locally from these Dockerfiles

**Current Role**: The Dockerfiles serve as:
- **Documentation**: Show how the images were originally built
- **Maintenance Tools**: For rebuilding images when updates are needed  
- **Reference Material**: Understanding what's inside the pre-built images

**Practical Impact**: You could delete these Dockerfiles and the devcontainer would still work perfectly, pulling the same pre-built images from Docker Hub. They are **historical artifacts** rather than active components of the daily devcontainer experience.

### Setup Scripts

#### entrypoint.sh
- **Purpose**: Container initialization and VNC setup
- **Functions**:
  - Creates custom user with sudo privileges
  - Sets up VNC server with password authentication
  - Configures desktop environment (MATE session)
  - Sets up supervisor for process management
  - Configures ROS environment in bashrc
  - Creates desktop shortcuts

#### setup_container.bash
- **Purpose**: Development environment setup
- **Functions**:
  - Sources ROS Jazzy environment
  - Creates ubuntu user with sudo access
  - Downloads and installs FreeCAD AppImage
  - Clones and installs FreeCAD Cross plugin
  - Installs Python dependencies (black, urdf-parser-py)
  - Clones robotics repositories (gazebosim tools)
  - Builds ROS workspace with colcon
  - Configures environment variables for Gazebo

### Installation Scripts

#### install_dependencies.sh
- Additional dependency installation script
- Handles package-specific requirements

### Container Access
- **VNC Access**: Available on port 6080 via web browser
- **Desktop Environment**: Full MATE desktop with terminal access
- **Default Password**: `ubuntu` (configurable)

## GitHub Actions Workflows

### .github/workflows/build.yaml
- **Name**: Build Test
- **Trigger**: Every push to any branch
- **Purpose**: Ensures code compiles successfully
- **Environment**: Ubuntu latest with ROS Jazzy
- **Steps**:
  1. Checkout code
  2. Setup ROS Jazzy environment
  3. Install dependencies via rosdep
  4. Configure Gazebo environment paths
  5. Build `krytn` package with testing enabled
- **Build Options**: Symlink install, merge install, testing enabled

### .github/workflows/test.yaml
- **Name**: ROS2 Tests
- **Trigger**: Push and pull requests to `jazzy` branch
- **Purpose**: Runs automated tests for ROS packages
- **Environment**: Ubuntu latest with ROS Jazzy
- **Steps**:
  1. Checkout code
  2. Setup ROS Jazzy environment
  3. Install dependencies
  4. Configure Gazebo paths (includes workspace install paths)
  5. Build and test `krytn` package
- **Test Options**: Package-specific testing, console output handlers, fail-fast disabled

### Key Features
- **Target Package**: Both workflows focus on the `krytn` package
- **ROS Distribution**: ROS Jazzy (ROS2)
- **Simulation**: Gazebo simulation environment configured
- **Testing**: Automated testing with detailed console output
- **CI/CD**: Continuous integration on code changes

## Architecture Summary

This project provides a complete ROS2 Jazzy development environment accessible via:
1. **Local Development**: DevContainer with VNC desktop access
2. **Automated Testing**: GitHub Actions for build verification and testing
3. **Multi-Platform**: Support for Ubuntu, WSL, and general Linux environments
4. **Robotics Tools**: Pre-configured with FreeCAD, Gazebo, and ROS development tools

The setup enables full robotics development workflow from design (FreeCAD) to simulation (Gazebo) to deployment (ROS2).