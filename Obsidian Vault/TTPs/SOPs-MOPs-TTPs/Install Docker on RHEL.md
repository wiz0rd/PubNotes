# Docker Engine Installation on RHEL TTP

## Table of Contents
1. [[#Prerequisites]]
2. [[#Uninstall Old Versions]]
3. [[#Installation Methods]]
   3.1. [[#Install using the rpm repository]]
   3.2. [[#Install from a package]]
   3.3. [[#Install using the convenience script]]
4. [[#Upgrade Docker Engine]]
5. [[#Uninstall Docker Engine]]
6. [[#Next Steps]]
7. [[#Links]]

## Prerequisites

- Maintained version of RHEL 8 or RHEL 9
- x86_64 or aarch64 architecture

## Uninstall Old Versions

```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine \
                podman \
                runc
```

## Installation Methods

### Install using the rpm repository

1. Set up the repository:
   ```bash
   sudo yum install -y yum-utils
   sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
   ```

2. Install Docker Engine:
   ```bash
   sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

3. Start Docker:
   ```bash
   sudo systemctl start docker
   ```

4. Verify installation:
   ```bash
   sudo docker run hello-world
   ```

### Install from a package

1. Download RPM files from https://download.docker.com/linux/rhel/
2. Install Docker Engine:
   ```bash
   sudo yum install ./containerd.io-<version>.<arch>.rpm \
     ./docker-ce-<version>.<arch>.rpm \
     ./docker-ce-cli-<version>.<arch>.rpm \
     ./docker-buildx-plugin-<version>.<arch>.rpm \
     ./docker-compose-plugin-<version>.<arch>.rpm
   ```

3. Start Docker:
   ```bash
   sudo systemctl start docker
   ```

4. Verify installation:
   ```bash
   sudo docker run hello-world
   ```

### Install using the convenience script

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Upgrade Docker Engine

- For repository installation: Follow the installation steps, choosing the new version.
- For package installation: Download new packages and repeat installation with `yum -y upgrade`.
- After convenience script: Use package manager directly.

## Uninstall Docker Engine

1. Remove packages:
   ```bash
   sudo yum remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   ```

2. Delete all images, containers, and volumes:
   ```bash
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
   ```

## Next Steps

- Configure Docker to start on boot
- Create a docker group and add your user
- Explore Docker documentation for further configuration and usage

## Links

- [Docker Engine on RHEL documentation](https://docs.docker.com/engine/install/rhel/)
- [Docker Desktop for Linux Early Access Program](https://www.docker.com/products/docker-desktop/)
- [Docker Engine release notes](https://docs.docker.com/engine/release-notes/)
- [Docker rootless mode documentation](https://docs.docker.com/engine/security/rootless/)
