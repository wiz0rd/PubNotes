# ContainerLab Setup Guide

## Index
1. [[Introduction]]
2. [[Prerequisites]]
3. [[Remove Podman]]
4. [[Install Docker]]
5. [[Install ContainerLab]]
6. [[Import Docker Images]]
7. [[Directory Structure]]
8. [[Create Topology File]]
9. [[Deploy Topology]]
10. [[Documentation Links]]

## Introduction

This guide will walk you through the process of setting up ContainerLab, a powerful tool for network topology simulation. We'll cover everything from removing conflicting software to deploying your first network topology.

## Prerequisites

- Red Hat Enterprise Linux 8 (RHEL 8) or compatible system
- Root or sudo access
- Internet connection for downloading packages

## Remove Podman

If Podman is installed, remove it to avoid conflicts with Docker:

```bash
sudo dnf remove podman buildah
```

## Install Docker

1. Remove any older versions of Docker:

```bash
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

2. Add the Docker CE repository:

```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

3. Install required dependencies:

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

4. Install Docker CE:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io
```

5. Start and enable the Docker service:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

6. Verify the installation:

```bash
sudo docker run hello-world
```

7. (Optional) Add your user to the docker group:

```bash
sudo usermod -aG docker your_username
```

Log out and log back in for the changes to take effect.

## Install ContainerLab

Install ContainerLab using the official installation script:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verify the installation:

```bash
containerlab version
```

## Import Docker Images

Assuming you have the .tar files for the images, use the following commands to import them:

```bash
docker load < ceos.tar
docker load < srlinux.tar
docker load < vjunos.tar
docker load < vr-c8000v.tar
docker load < vr-csr.tar
docker load < vr-ftdv.tar
docker load < vr-n9kv.tar
docker load < vr-xrv.tar
```

Verify the imported images:

```bash
docker images
```

## Directory Structure

Create a directory structure for your ContainerLab projects:

```
containerlab/
├── labs/
│   ├── three-router-topology/
│   │   ├── topology.yml
│   │   └── configs/
│   │       ├── vjunos-switch/
│   │       ├── csr-router/
│   │       └── n9kv-switch/
│   └── other-topologies/
└── images/
    ├── ceos.tar
    ├── srlinux.tar
    ├── vjunos.tar
    └── ...
```

Create this structure:

```bash
mkdir -p containerlab/labs/three-router-topology/configs/{vjunos-switch,csr-router,n9kv-switch}
mkdir -p containerlab/images
```

## Create Topology File

Create a `topology.yml` file in the `containerlab/labs/three-router-topology/` directory:

```yaml
name: three-router-topology
mgmt:
  network: three-router-mgmt
  ipv4-subnet: 172.100.100.0/24
topology:
  nodes:
    vjunos-switch:
      kind: juniper_vjunosswitch
      image: vrnetlab/vr-vjunosswitch:23.2R1.14
      mgmt-ipv4: 172.100.100.10
    csr-router:
      kind: vr-csr
      image: vrnetlab/vr-csr:17.03.08
      mgmt-ipv4: 172.100.100.20
    n9kv-switch:
      kind: vr-n9kv
      image: vrnetlab/vr-n9kv:10.3.5
      mgmt-ipv4: 172.100.100.30
  links:
    - endpoints: ["vjunos-switch:eth1", "csr-router:eth1"]
    - endpoints: ["vjunos-switch:eth2", "csr-router:eth2"]
    - endpoints: ["vjunos-switch:eth3", "n9kv-switch:eth1"]
    - endpoints: ["vjunos-switch:eth4", "n9kv-switch:eth2"]
    - endpoints: ["csr-router:eth3", "n9kv-switch:eth3"]
    - endpoints: ["csr-router:eth4", "n9kv-switch:eth4"]
```

## Deploy Topology

Navigate to the topology directory and deploy:

```bash
cd containerlab/labs/three-router-topology/
containerlab deploy -t topology.yml
```

## Documentation Links

- [Docker Documentation](https://docs.docker.com/)
- [ContainerLab Documentation](https://containerlab.dev/manual/getting-started/)
- [ContainerLab Topology Definition](https://containerlab.dev/manual/topo-def-file/)
- [ContainerLab Node Types](https://containerlab.dev/manual/kinds/overview/)
