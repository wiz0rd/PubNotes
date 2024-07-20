# Standard Operating Procedure: Setting Up a Restricted SFTP Server

## Index
1. [[#1. Introduction]]
2. [[#2. Prerequisites]]
3. [[#3. Installation]]
4. [[#4. User and Directory Setup]]
5. [[#5. SSH Configuration]]
6. [[#6. File Management]]
7. [[#7. Verification]]
8. [[#8. Security Considerations]]
9. [[#9. Troubleshooting]]
10. [[#10. Maintenance]]
11. [[#11. Links and References]]

## 1. Introduction

This SOP outlines the process of setting up a restricted SFTP server on a Red Hat system. The server will be configured with a service account that only has access to download files from a specific directory.

## 2. Prerequisites

- Red Hat Enterprise Linux (RHEL) or CentOS system
- Root or sudo access
- Internet connectivity for package installation

## 3. Installation

### 3.1 Install OpenSSH Server

If not already installed, install the OpenSSH server:

```bash
sudo dnf install openssh-server
```

## 4. User and Directory Setup

### 4.1 Create SFTP User

Create a dedicated user for SFTP access:

```bash
sudo useradd -m sftpuser
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

```text
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

## 7. Verification

### 7.1 Test SFTP Access

Test the SFTP server by connecting from another machine:

```bash
sftp sftpuser@your_server_ip
```

Try downloading a file from the /download directory.

## 8. Security Considerations

- Regularly update the system and OpenSSH server.
- Use strong passwords or consider implementing SSH key authentication.
- Monitor SFTP logs for any suspicious activity.
- Implement firewall rules to restrict SFTP access if necessary.

## 9. Troubleshooting

- If unable to connect, check the SSH service status: `sudo systemctl status sshd`
- Review SSH logs for error messages: `sudo journalctl -u sshd`
- Verify file permissions in the SFTP directory structure.

## 10. Maintenance

- Regularly review and remove unnecessary files from the download directory.
- Periodically audit user access and remove unused accounts.
- Keep the SSH configuration up to date with current security best practices.

## 11. Links and References

- [OpenSSH Official Documentation](https://www.openssh.com/manual.html)
- [Red Hat Enterprise Linux 8 - Configuring OpenSSH](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/assembly_using-secure-communications-between-two-systems-with-openssh_securing-networks)
- [SFTP Security Best Practices](https://www.ssh.com/academy/ssh/sftp/security)
- [Linux File Permissions Explained](https://linuxize.com/post/understanding-linux-file-permissions/)
- [SELinux User's and Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index)
