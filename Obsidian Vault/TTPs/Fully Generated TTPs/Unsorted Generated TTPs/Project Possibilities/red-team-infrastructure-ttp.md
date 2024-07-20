# Red Team Infrastructure TTP

## Project Goal
The goal of this project is to set up a robust, covert red team infrastructure for conducting penetration testing and simulated attacks. By the end of this project, you will have a secure and stealthy network of systems that can be used for red team operations.

## Overview
This TTP provides detailed instructions for setting up a red team infrastructure, including command and control (C2) systems, redirectors, and various operational security measures.

![Red Team Infrastructure Overview](https://example.com/path/to/red_team_infrastructure.png)
*Figure 1: Red Team Infrastructure Overview. Replace with an actual diagram showing the components of your red team infrastructure.*

## Prerequisites
- Basic understanding of networking and security concepts
- Familiarity with Linux systems administration
- Knowledge of penetration testing methodologies
- Multiple VPS providers or cloud accounts for distributed infrastructure
- Domain names for C2 servers and redirectors

## Warning
Ensure you have proper authorization before setting up and using red team infrastructure. Unauthorized use may be illegal and unethical.

## Step-by-step Guide

### 1. Set Up Domain Infrastructure

1. Purchase domain names from a privacy-respecting registrar
2. Set up DNS records for your domains

```bash
# Example DNS configuration
A     example.com       203.0.113.1
CNAME cdn.example.com   example.com
```

### 2. Set Up Redirectors

1. Spin up a VPS for your redirector
2. Install and configure Nginx:

```bash
sudo apt update
sudo apt install nginx
```

3. Configure Nginx as a reverse proxy (in `/etc/nginx/sites-available/default`):

```nginx
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass http://YOUR_C2_SERVER_IP;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

4. Enable the configuration and restart Nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

![Nginx Configuration](https://example.com/path/to/nginx_config.png)
*Figure 2: Nginx Configuration for Redirector. Replace with an actual screenshot of your Nginx configuration.*

### 3. Set Up Command and Control (C2) Server

1. Spin up a VPS for your C2 server
2. Install Cobalt Strike (or your preferred C2 framework):

```bash
# Example for Cobalt Strike (replace with actual licensed version)
wget https://example.com/cobaltstrike-dist.tgz
tar -xzvf cobaltstrike-dist.tgz
```

3. Generate a malleable C2 profile:

```bash
./c2lint profile.profile
```

4. Start the Team Server:

```bash
./teamserver <C2_IP> <password> profile.profile
```

![Cobalt Strike Team Server](https://example.com/path/to/cobaltstrike_teamserver.png)
*Figure 3: Cobalt Strike Team Server. Replace with an actual screenshot of your C2 server setup (ensure no sensitive information is visible).*

### 4. Implement HTTPS Encryption

1. Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtain and configure SSL certificates:

```bash
sudo certbot --nginx -d example.com
```

### 5. Set Up Operational Logging

1. Install and configure rsyslog:

```bash
sudo apt install rsyslog
```

2. Configure remote logging in `/etc/rsyslog.conf`:

```
*.* @@log.example.com:514
```

3. Restart rsyslog:

```bash
sudo systemctl restart rsyslog
```

### 6. Implement SSH Hardening

1. Edit SSH configuration (`/etc/ssh/sshd_config`):

```
PermitRootLogin no
PasswordAuthentication no
UsePAM no
```

2. Restart SSH service:

```bash
sudo systemctl restart ssh
```

### 7. Set Up a VPN for Secure Access

1. Install OpenVPN:

```bash
sudo apt install openvpn
```

2. Configure OpenVPN server (use openvpn-install script for simplicity)

3. Generate and distribute client certificates

### 8. Implement Traffic Obfuscation

1. Set up domain fronting or use Cloudflare as a proxy

2. Implement protocol obfuscation (e.g., using obfs4)

### 9. Create Operational Security Policies

1. Develop and document OPSEC procedures
2. Implement need-to-know principles for team members
3. Establish secure communication channels for the team

### 10. Conduct Infrastructure Testing

1. Perform external scans to ensure stealthiness
2. Test all components of the infrastructure
3. Conduct a simulated operation to verify functionality

![Infrastructure Test Results](https://example.com/path/to/infrastructure_test.png)
*Figure 4: Infrastructure Test Results. Replace with an actual screenshot of your infrastructure test results (ensure no sensitive information is visible).*

## Advanced Topics

- Implementing cloud-based redirection (e.g., AWS Lambda, Azure Functions)
- Using Tor hidden services for additional anonymity
- Implementing custom implants and payloads
- Leveraging cloud storage for data exfiltration
- Implementing SDN for dynamic infrastructure management

## Troubleshooting

- If C2 communication fails, check firewall rules and redirector configurations
- For SSL issues, verify certificate validity and configuration
- If experiencing detection, review and modify C2 profiles and traffic patterns
- Use network analysis tools (e.g., Wireshark) to diagnose communication issues

## Best Practices

1. Regularly rotate infrastructure components and IP addresses
2. Use separate infrastructure for each engagement
3. Implement strict access controls and authentication measures
4. Regularly update and patch all systems
5. Maintain detailed documentation of your infrastructure
6. Conduct regular security assessments of your own infrastructure
7. Ensure compliance with all relevant laws and regulations

## Additional Resources

- [Red Team Infrastructure Wiki](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki)
- [Cobalt Strike Documentation](https://www.cobaltstrike.com/help-overview)
- [The Hacker Playbook 3: Practical Guide To Penetration Testing](https://www.amazon.com/Hacker-Playbook-Practical-Penetration-Testing/dp/1980901759)
- [SANS SEC564: Red Team Operations and Threat Emulation](https://www.sans.org/cyber-security-courses/red-team-operations-and-threat-emulation/)

## Glossary

- **Red Team**: A group authorized and organized to emulate a potential adversary's attack or exploitation capabilities against an enterprise's security posture
- **C2 (Command and Control)**: Infrastructure used to maintain communication with compromised systems within a target network
- **OPSEC (Operations Security)**: A process that identifies critical information to determine if friendly actions can be observed by enemy intelligence
- **Redirector**: A server that forwards traffic to hide the true location of the C2 server
- **Domain Fronting**: A technique to obscure the true endpoint of a connection
- **Malleable C2**: A domain-specific language to change the network indicators in Cobalt Strike
- **Implant**: A piece of software that runs on a compromised system and communicates with the C2 server
- **Exfiltration**: The unauthorized transfer of data from a computer or server

