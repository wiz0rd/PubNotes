# Standard Operating Procedure: Setting Up ContainerLab Environment with Podman on Red Hat

## Index
1. [[#1. Introduction]]
2. [[#2. Prerequisites]]
3. [[#3. User and Directory Setup]]
4. [[#4. Python Installation and Virtual Environment Setup]]
5. [[#5. Package Installation and Configuration]]
6. [[#6. Podman Setup]]
7. [[#7. ContainerLab Configuration]]
8. [[#8. Image Management]]
9. [[#9. Using ContainerLab with Podman]]
10. [[#10. Verification]]
11. [[#11. Troubleshooting]]
12. [[#12. Maintenance]]
13. [[#13. Security Considerations]]
14. [[#14. Links and References]]

## 1. Introduction

This SOP outlines the process of setting up a ContainerLab environment using Podman on a Red Hat system. This setup allows for testing network topologies and configurations in a containerized environment.

## 2. Prerequisites

- Red Hat Enterprise Linux (RHEL) or CentOS system
- Root or sudo access
- Internet connectivity for package installation

## 3. User and Directory Setup

### 3.1 Create Service Account

Create a service account with sudo privileges:

```bash
sudo useradd -m -d /home/svc.nmb.cm svc.nmb.cm
sudo passwd svc.nmb.cm
sudo usermod -aG wheel svc.nmb.cm
```

### 3.2 Create and Configure Working Directory

Set up the working directory:

```bash
sudo mkdir -p /home/nmb
sudo chown svc.nmb.cm:svc.nmb.cm /home/nmb
sudo chmod 775 /home/nmb
sudo ln -s /home/nmb /nmb
```

### 3.3 Add Users to Service Account Group

Add other users to the service account group (replace 'username' with actual username):

```bash
sudo usermod -aG svc.nmb.cm username
```

## 4. Python Installation and Virtual Environment Setup

### 4.1 Install Python 3.12

```bash
sudo dnf install python3.12 python3.12-devel
```

### 4.2 Create Virtual Environment

```bash
mkdir -p /nmb/Environments
python3.12 -m venv /nmb/Environments/ContainerLab
```

### 4.3 Activate Virtual Environment

```bash
source /nmb/Environments/ContainerLab/bin/activate
```

## 5. Package Installation and Configuration

### 5.1 Upgrade pip

```bash
pip install --upgrade pip
```

### 5.2 Install Ansible

```bash
pip install ansible
```

### 5.3 Install Ansible Galaxy Collections

```bash
ansible-galaxy collection install cisco.ios cisco.nxos cisco.asa juniper.junos
```

### 5.4 Install Supporting Python Packages

```bash
pip install ncclient pyyaml paramiko netmiko napalm jinja2 pylibssh
```

### 5.5 Install ContainerLab

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

## 6. Podman Setup

### 6.1 Install Podman

```bash
sudo dnf install podman
```

### 6.2 Configure Podman for Rootless Operation

```bash
sudo systemctl enable --now podman.socket
```

## 7. ContainerLab Configuration

### 7.1 Create ContainerLab Configuration File

```bash
sudo mkdir -p /etc/containerlab
sudo nano /etc/containerlab/config.yml
```

Add the following content:

```yaml
runtime:
  name: podman
```

## 8. Image Management

### 8.1 Pull Necessary Images

Example with Alpine:

```bash
podman pull alpine:latest
```

### 8.2 Verify Imported Images

```bash
podman images
```

### 8.3 Import Docker Images to Podman

If you have Docker images exported as tar files:

```bash
podman load -i image_name.tar
```

## 9. Using ContainerLab with Podman

When running ContainerLab commands, use the `--runtime podman` flag to specify Podman as the runtime. This tells ContainerLab to use Podman instead of Docker for container operations.

Example:

```bash
sudo containerlab deploy --runtime podman -t your_topology_file.yml
```

You can also set the `CLAB_RUNTIME` environment variable to avoid specifying the runtime flag in each command:

```bash
export CLAB_RUNTIME=podman
sudo containerlab deploy -t your_topology_file.yml
```

Note: The `sudo` is necessary because ContainerLab typically requires root privileges to set up network namespaces and interfaces.

## 10. Verification

### 10.1 Verify Podman Operation

```bash
podman run --rm hello-world
```

### 10.2 Verify ContainerLab Operation

```bash
sudo containerlab deploy --runtime podman -t your_topology_file.yml
```

## 11. Troubleshooting

- If Podman fails to start containers, check system resources and Podman logs.
- For ContainerLab issues, verify the topology file and ensure all required images are available.
- If you encounter permission issues, make sure you're using `sudo` with ContainerLab commands.
- Check ContainerLab logs for detailed error messages: `sudo containerlab inspect --all`

## 12. Maintenance

- Regularly update Podman and ContainerLab to their latest versions.
- Keep container images up to date.
- Periodically clean up unused containers and images to free up disk space:
  ```bash
  podman system prune
  ```

## 13. Security Considerations

- Use rootless Podman when possible to minimize security risks.
- Regularly apply system updates and security patches.
- Implement proper access controls for the ContainerLab environment.
- Monitor system logs for any suspicious activities.
- Be cautious when running ContainerLab with sudo privileges.

## 14. Links and References

- [ContainerLab Official Documentation](https://containerlab.dev/)
- [ContainerLab with Podman Guide](https://containerlab.dev/install/#podman)
- [Podman Official Documentation](https://docs.podman.io/en/latest/)
- [Red Hat Enterprise Linux 8 - Working with Containers](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/index)
- [Ansible Documentation](https://docs.ansible.com/)
- [Python Virtual Environments Guide](https://docs.python.org/3/tutorial/venv.html)
- [Network Automation with ContainerLab](https://containerlab.dev/lab-examples/network-automation/)
- [Podman Security Guide](https://docs.podman.io/en/latest/markdown/podman-seccomp.1.html)
