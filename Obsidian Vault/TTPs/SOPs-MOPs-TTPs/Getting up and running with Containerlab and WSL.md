# Comprehensive Guide: Containerlab and Ansible with vrnetlab on WSL Ubuntu

## Project Statement

This project aims to set up a network simulation environment using Containerlab and Ansible on Windows Subsystem for Linux (WSL) Ubuntu. We'll use vrnetlab to generate container images for Cisco CSR 1000v, Nexus 9000v, and Juniper vSRX. The final goal is to launch a three-router topology with two links between each router, providing a realistic network simulation for testing and development purposes.

## Prerequisites

- Windows 10 (version 1903 or higher, Build 18362 or higher)
- WSL 2 enabled
- Ubuntu installed from the Microsoft Store
- Basic familiarity with command-line interfaces
- Cisco and Juniper image files (to be obtained separately due to licensing requirements)

## Table of Contents

1. [[#1 Setting up WSL and Ubuntu|Setting up WSL and Ubuntu]]
2. [[#2 Installing Docker in Ubuntu on WSL|Installing Docker in Ubuntu on WSL]]
3. [[#3 Installing Containerlab|Installing Containerlab]]
4. [[#4 Setting up vrnetlab|Setting up vrnetlab]]
5. [[#5 Obtaining and Converting Network Images|Obtaining and Converting Network Images]]
6. [[#6 Installing Ansible|Installing Ansible]]
7. [[#7 Creating a Three-Router Topology|Creating a Three-Router Topology]]
8. [[#8 Launching and Managing the Topology|Launching and Managing the Topology]]

## 1. Setting up WSL and Ubuntu

If you haven't already set up WSL and Ubuntu, follow these steps:

1. Open PowerShell as Administrator and run:
   ```powershell
   wsl --install
   ```
   This command installs WSL with Ubuntu as the default distribution.

2. Restart your computer when prompted.

3. After reboot, Ubuntu will start automatically. Set up your username and password when prompted.

4. Update and upgrade your Ubuntu installation:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

## 2. Installing Docker in Ubuntu on WSL

Follow these steps to install Docker in Ubuntu on WSL:

1. Enable WSL 2:
   Open PowerShell as an Administrator and run:
   ```powershell
   wsl --set-default-version 2
   ```

2. Install Docker Desktop:
   Download and install Docker Desktop for Windows from the official Docker website. Ensure that you select the option to use the WSL 2 based engine during installation.

3. Configure Docker Desktop:
   After installation, open Docker Desktop and go to **Settings** > **General**. Ensure that "Use the WSL 2 based engine" is checked.
   Next, go to **Settings** > **Resources** > **WSL Integration**. Enable integration with your Ubuntu distribution by checking the corresponding box.

4. Install Docker in WSL:
   Open your Ubuntu terminal (WSL) and run the following commands:
   ```bash
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt update
   sudo apt install docker-ce
   ```

5. Configure Docker to start on boot:
   ```bash
   sudo systemctl enable docker
   ```

6. Start Docker:
   ```bash
   sudo service docker start
   ```

7. Verify Docker installation:
   ```bash
   docker --version
   ```

8. Test Docker installation:
   ```bash
   sudo docker run hello-world
   ```

9. Add your user to the Docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Log out and log back in to apply the changes.

## 3. Installing Containerlab

You have two options for installing Containerlab:

### Option 1: Install Containerlab with all dependencies

This option installs Containerlab along with all its dependencies, including Docker:

```bash
curl -sL https://containerlab.dev/setup | sudo bash -s "all"
```

### Option 2: Install only Containerlab

If you already have Docker installed and only want to install Containerlab:

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

After installation, verify it by running:

```bash
containerlab version
```

## 4. Setting up vrnetlab

We'll use a specific fork of vrnetlab that's compatible with Containerlab. This version is maintained by Roman Dodin (hellt) and includes some improvements for Containerlab integration.

1. Clone the hellt/vrnetlab repository:
   ```bash
   git clone https://github.com/hellt/vrnetlab.git
   cd vrnetlab
   ```

2. Install required packages:
   ```bash
   sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
   ```

## 5. Obtaining and Converting Network Images

Due to licensing restrictions, you need to obtain the network OS images separately. Once you have the images, follow these steps to convert them:

### For Cisco CSR 1000v:

1. Place the CSR1000v image in the `vrnetlab/csr` directory.
2. Build the Docker image:
   ```bash
   cd vrnetlab/csr
   make docker-image
   ```

### For Cisco Nexus 9000v:

1. Place the Nexus 9000v image in the `vrnetlab/nxos` directory.
2. Build the Docker image:
   ```bash
   cd vrnetlab/nxos
   make docker-image
   ```

### For Juniper vSRX:

1. Place the vSRX image in the `vrnetlab/vr-vsrx` directory.
2. Build the Docker image:
   ```bash
   cd vrnetlab/vr-vsrx
   make docker-image
   ```

## 6. Installing Ansible

Ansible will be used for configuration management. To install Ansible:

1. Install Ansible using pip:
   ```bash
   sudo apt install python3-pip
   pip3 install ansible
   ```

2. Verify the installation:
   ```bash
   ansible --version
   ```

## 7. Creating a Three-Router Topology

Create a Containerlab topology file named `three_router_topology.yml`:

```yaml
name: three-router-topology

topology:
  nodes:
    router1:
      kind: vr-csr
      image: vrnetlab/vr-csr:latest
    router2:
      kind: vr-n9kv
      image: vrnetlab/vr-n9kv:latest
    router3:
      kind: vr-vsrx
      image: vrnetlab/vr-vsrx:latest

  links:
    - endpoints: ["router1:eth1", "router2:eth1"]
    - endpoints: ["router1:eth2", "router3:eth1"]
    - endpoints: ["router2:eth2", "router3:eth2"]
    - endpoints: ["router1:eth3", "router2:eth3"]
    - endpoints: ["router1:eth4", "router3:eth3"]
    - endpoints: ["router2:eth4", "router3:eth4"]
```

## 8. Launching and Managing the Topology

1. Deploy the topology:
   ```bash
   sudo containerlab deploy -t three_router_topology.yml
   ```

2. Verify the deployment:
   ```bash
   sudo containerlab inspect -t three_router_topology.yml
   ```

3. To access a router's console:
   ```bash
   sudo docker exec -it clab-three-router-topology-router1 /bin/bash
   ```
   Replace `router1` with `router2` or `router3` as needed.

4. To stop and remove the topology:
   ```bash
   sudo containerlab destroy -t three_router_topology.yml
   ```

This completes the setup of your Containerlab and Ansible environment with vrnetlab-generated containers on WSL Ubuntu. You now have a three-router topology with two links between each router, ready for further configuration and testing.
