# Installing Docker in Ubuntu on WSL

## Overview

This document provides a step-by-step guide to installing Docker in Ubuntu on Windows Subsystem for Linux (WSL). Follow these instructions to set up a functional Docker environment on your WSL installation.

## Prerequisites

- Windows 10, version 1903 or higher (Build 18362 or higher)
- WSL 2 enabled
- Ubuntu installed from the Microsoft Store

## Steps

### 1. Enable WSL 2

Ensure that WSL 2 is enabled on your system. Open PowerShell as an Administrator and run the following commands:

```powershell
wsl --set-default-version 2
```

### 2. Install Docker Desktop

Download and install Docker Desktop for Windows from the official Docker website. Ensure that you select the option to use the WSL 2 based engine during installation.

### 3. Configure Docker Desktop

After installation, open Docker Desktop and go to **Settings** > **General**. Ensure that "Use the WSL 2 based engine" is checked.

Next, go to **Settings** > **Resources** > **WSL Integration**. Enable integration with your Ubuntu distribution by checking the corresponding box.

### 4. Install Docker in WSL

Open your Ubuntu terminal (WSL) and update your package list:
```
sudo apt update
```

Install Docker's package dependencies:
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Add Docker's APT repository:
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
Update your package list again:
```
sudo apt update
```

Install Docker:
```
sudo apt install docker-ce
```

### 5. Configure Docker to Start on Boot

Enable the Docker service to start on boot:
```
sudo systemctl enable docker
```

6. Start Docker

Start the Docker service:
```
sudo service docker start
```

### 7. Verify Docker Installation

Verify that Docker is installed correctly by running:
```
docker --version
```
You should see the Docker version information.

### 8. Test Docker Installation

Run a test Docker container to ensure everything is set up correctly:
```
sudo docker run hello-world
```
You should see a message indicating that Docker is working correctly.

### 9. Add Your User to the Docker Group

To avoid typing `sudo` whenever you run the `docker` command, add your user to the `docker` group:
```
sudo usermod -aG docker $USER
```
After running this command, log out and log back in to apply the changes.

## Conclusion

You have successfully installed Docker in Ubuntu on WSL. You can now use Docker to manage containers directly from your WSL environment.

