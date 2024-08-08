# Technical Task Procedure: Setting Up a Restricted SFTP Server

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Installation](#3-installation)
4. [User and Directory Setup](#4-user-and-directory-setup)
5. [SSH Configuration](#5-ssh-configuration)
6. [File Management](#6-file-management)
7. [Verification](#7-verification)
8. [Security Considerations](#8-security-considerations)
9. [Troubleshooting](#9-troubleshooting)
10. [Maintenance](#10-maintenance)
11. [Links and References](#11-links-and-references)

## 1. Introduction

This Technical Task Procedure (TTP) outlines the process of setting up a restricted SFTP server on a Red Hat Enterprise Linux 8 (RHEL 8) system. The server will be configured with a service account that only has access to download files from a specific directory. This setup is designed to provide network administrators with easy access to the latest approved images while maintaining strict security measures.

## 2. Prerequisites

- Red Hat Enterprise Linux 8 (RHEL 8) or CentOS 8 system
- Root or sudo access
- Internet connectivity for package installation

## 3. Installation

### 3.1 Install OpenSSH Server

If not already installed, install the OpenSSH server:

```bash
sudo dnf install openssh-server
```

### 3.2 Enable and Start the SSH Service

Ensure the SSH service is enabled and started:

```bash
sudo systemctl enable sshd
sudo systemctl start sshd
```

## 4. User and Directory Setup

### 4.1 Create SFTP User

Create a dedicated user for SFTP access with a non-interactive shell:

```bash
sudo useradd -m -s /sbin/nologin sftpuser
sudo passwd sftpuser
```

### 4.2 Create and Configure SFTP Directory

Set up a directory structure for SFTP:

```bash
sudo mkdir -p /home/sftpuser/download
sudo chown root:root /home/sftpuser
sudo chmod 755 /home/sftpuser
sudo chown sftpuser:sftpuser /home/sftpuser/download
sudo chmod 755 /home/sftpuser/download
```

## 5. SSH Configuration

### 5.1 Configure SFTP-only Access

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following content at the end of the file:

```
Match User sftpuser
    ChrootDirectory /home/sftpuser
    ForceCommand internal-sftp
    PasswordAuthentication yes
    PermitTunnel no
    AllowAgentForwarding no
    AllowTcpForwarding no
    X11Forwarding no
```

### 5.2 Restart SSH Service

Apply the changes by restarting the SSH service:

```bash
sudo systemctl restart sshd
```

## 6. File Management

### 6.1 Adding Files for Download

To add files for the SFTP user to download:

1. Switch to a user with sudo privileges.
2. Copy files to the download directory:

```bash
sudo cp /path/to/file /home/sftpuser/download/
```

3. Set appropriate permissions:

```bash
sudo chown sftpuser:sftpuser /home/sftpuser/download/filename
sudo chmod 644 /home/sftpuser/download/filename
```

### 6.2 Creating Symlinks

Create symlinks to point to the latest approved images:

```bash
sudo ln -s /home/sftpuser/download/actual_file.img /home/sftpuser/download/latest
```

## 7. Verification

### 7.1 Test SFTP Access

Test the SFTP server by connecting from another machine:

```bash
sftp sftpuser@your_server_ip
```

Try downloading a file from the `/download` directory.

### 7.2 Verify Restrictions

Attempt the following actions to ensure restrictions are in place:

- Try to upload a file (should be denied)
- Try to move outside the `/download` directory (should be denied)
- Attempt SSH access (should be denied)

## 8. Security Considerations

- Regularly update the system and OpenSSH server
- Use strong passwords or consider implementing SSH key authentication
- Monitor SFTP logs for any suspicious activity
- Implement firewall rules to restrict SFTP access if necessary

### 8.1 Firewall Configuration

Configure the firewall to allow SSH traffic (which includes SFTP):

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### 8.2 SELinux Configuration

If SELinux is enabled, set the appropriate context for the SFTP root:

```bash
sudo semanage fcontext -a -t ssh_home_t "/home/sftpuser(/.*)?"
sudo restorecon -R -v /home/sftpuser
```

## 9. Troubleshooting

- If unable to connect, check the SSH service status:
  ```bash
  sudo systemctl status sshd
  ```
- Review SSH logs for error messages:
  ```bash
  sudo journalctl -u sshd
  ```
- Verify file permissions in the SFTP directory structure

## 10. Maintenance

### 10.1 Regular Tasks

- Regularly review and remove unnecessary files from the download directory
- Periodically audit user access and remove unused accounts
- Keep the SSH configuration up to date with current security best practices

### 10.2 Updating Symlinks

Create a bash script (`update_symlinks.sh`) to automate symlink updates:

```bash
#!/bin/bash

IMAGE_TYPES=("Juniper" "Cisco/IOS" "Cisco/NX-OS")

for type in "${IMAGE_TYPES[@]}"; do
    latest_image=$(ls -t /home/sftpuser/download/$type/*.img | head -n1)
    if [ -n "$latest_image" ]; then
        ln -sf "$latest_image" "/home/sftpuser/download/$type/latest"
        echo "Updated symlink for $type"
    else
        echo "No images found for $type"
    fi
done
```

Set up a daily cron job to run this script:

```bash
sudo crontab -e
# Add the following line:
0 1 * * * /path/to/update_symlinks.sh
```

## 11. Links and References

- [OpenSSH Official Documentation](https://www.openssh.com/manual.html)
- [Red Hat Enterprise Linux 8 - Configuring OpenSSH](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-secure-communications-between-two-systems-with-openssh_securing-networks)
- [SFTP Security Best Practices](https://www.ssh.com/academy/ssh/sftp/security)
- [Linux File Permissions Explained](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/)
- [SELinux User's and Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/index)
