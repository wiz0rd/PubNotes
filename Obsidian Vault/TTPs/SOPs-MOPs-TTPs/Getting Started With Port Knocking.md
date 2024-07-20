# SSH Port Knocking: Setup and Best Practices

## Table of Contents

1. [[#Introduction]]
2. [[#What is Port Knocking?]]
3. [[#Benefits of Port Knocking]]
4. [[#Setting Up Port Knocking]] 4.1. [[#Prerequisites]] 4.2. [[#Installation]] 4.3. [[#Configuration]] 4.4. [[#Firewall Rules]]
5. [[#Client-Side Configuration]]
6. [[#Best Practices]]
7. [[#Limitations and Considerations]]
8. [[#Troubleshooting]]
9. [[#Additional Resources]]

## Introduction

This document provides a comprehensive guide on setting up and using SSH port knocking to enhance the security of your devices against intrusion attempts.

## What is Port Knocking?

Port knocking is a method of externally opening ports on a firewall by generating a connection attempt on a set of prespecified closed ports. Once a correct sequence of connection attempts is received, the firewall opens certain port(s) to allow a connection.

## Benefits of Port Knocking

- Adds an extra layer of security to SSH connections
- Reduces exposure to automated scanning and brute-force attacks
- Conceals the SSH service from casual port scans

## Setting Up Port Knocking

### Prerequisites

- A Linux server with root access
- Basic knowledge of Linux command line and SSH

### Installation

1. Install knockd (the port knocking daemon):

   ```bash
   sudo apt-get update
   sudo apt-get install knockd
   ```

### Configuration

1. Edit the knockd configuration file:

   ```bash
   sudo nano /etc/knockd.conf
   ```

2. Add the following configuration:

   ```
   [options]
       UseSyslog

   [openSSH]
       sequence    = 7000,8000,9000
       seq_timeout = 5
       command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
       tcpflags    = syn

   [closeSSH]
       sequence    = 9000,8000,7000
       seq_timeout = 5
       command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
       tcpflags    = syn
   ```

### Firewall Rules

1. Configure your firewall to drop all incoming traffic to port 22 by default:

   ```bash
   sudo iptables -A INPUT -p tcp --dport 22 -j DROP
   ```

2. Allow the knockd daemon to listen on the knock ports:

   ```bash
   sudo iptables -A INPUT -p tcp --dport 7000 -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 8000 -j ACCEPT
   sudo iptables -A INPUT -p tcp --dport 9000 -j ACCEPT
   ```

3. Save the iptables rules:

   ```bash
   sudo iptables-save > /etc/iptables/rules.v4
   ```

## Client-Side Configuration

To knock on the server from a client machine:

1. Install knock:

   ```bash
   sudo apt-get install knock
   ```

2. Use the following command to open the SSH port:

   ```bash
   knock <server_ip> 7000 8000 9000
   ```

3. Connect via SSH as usual:

   ```bash
   ssh user@server_ip
   ```

4. Close the port after your session:

   ```bash
   knock <server_ip> 9000 8000 7000
   ```

## Best Practices

1. Use non-sequential ports for knocking
2. Implement a timeout for successful knocks
3. Combine port knocking with other security measures (e.g., key-based authentication, fail2ban)
4. Regularly change the knocking sequence
5. Use TCP SYN packets for knocking to avoid leaving traces in system logs

## Limitations and Considerations

- Port knocking is not foolproof and should be used as part of a layered security approach
- It can be vulnerable to replay attacks if not implemented carefully
- May introduce slight delays in connection establishment

## Troubleshooting

- Check knockd logs: `sudo journalctl -u knockd`
- Verify iptables rules: `sudo iptables -L`
- Ensure knockd is running: `sudo systemctl status knockd`

## Additional Resources

- [Port Knocking on Wikipedia](https://en.wikipedia.org/wiki/Port_knocking)
- [knockd Official Documentation](https://linux.die.net/man/1/knockd)
- [Iptables Tutorial](https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html)
- [SSH Security Best Practices](https://www.ssh.com/ssh/security/)

