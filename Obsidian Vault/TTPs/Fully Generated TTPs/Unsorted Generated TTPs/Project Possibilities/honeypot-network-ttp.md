# Honeypot Network TTP

## Project Goal
The goal of this project is to set up a network of honeypots to attract and analyze potential cyber attacks. By the end of this project, you will have a functioning honeypot network that can help you understand attacker behaviors, detect new types of attacks, and improve your overall security posture.

## Overview
This TTP provides detailed instructions for setting up a honeypot network using T-Pot, a multi-honeypot platform based on well-established honeypot daemons.

![Honeypot Network Overview](https://example.com/path/to/honeypot_network_overview.png)
*Figure 1: Honeypot Network Overview. Replace with an actual diagram showing the structure of your honeypot network.*

## Prerequisites
- A dedicated machine or virtual machine with at least 8GB RAM and 128GB disk space
- Ubuntu 22.04 LTS installed
- Basic understanding of network security concepts
- Familiarity with Linux command line
- Understanding of Docker containers

## Step-by-step Guide

### 1. Prepare the System

Update the system and install necessary packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl net-tools software-properties-common -y
```

### 2. Install T-Pot

1. Clone the T-Pot repository:

```bash
git clone https://github.com/telekom-security/tpotce
```

2. Change to the T-Pot directory:

```bash
cd tpotce/iso/installer/
```

3. Run the installation script:

```bash
sudo ./install.sh --type=user
```

4. Follow the prompts to configure T-Pot. Choose the type of installation (e.g., standard, sensor, industrial, etc.)

![T-Pot Installation](https://example.com/path/to/tpot_installation.png)
*Figure 2: T-Pot Installation Process. Replace with an actual screenshot of the T-Pot installation process.*

### 3. Configure Firewall

T-Pot will automatically configure iptables rules. Verify the rules:

```bash
sudo iptables -L
```

### 4. Start T-Pot Services

T-Pot services should start automatically after installation. If not, start them manually:

```bash
sudo systemctl start tpot
```

### 5. Access T-Pot Web Interface

1. Find your machine's IP address:

```bash
ip addr show
```

2. Access the web interface by navigating to `https://<your-ip-address>:64297` in a web browser

3. Log in using the credentials you set during installation

![T-Pot Web Interface](https://example.com/path/to/tpot_web_interface.png)
*Figure 3: T-Pot Web Interface. Replace with an actual screenshot of the T-Pot web interface.*

### 6. Configure Individual Honeypots

T-Pot includes various honeypots. Here's how to configure some popular ones:

#### Cowrie (SSH/Telnet honeypot)

1. Edit the Cowrie configuration:

```bash
sudo nano /opt/cowrie/cowrie.cfg
```

2. Modify settings like the hostname, SSH version, etc.

#### Dionaea (Multi-protocol honeypot)

1. Edit the Dionaea configuration:

```bash
sudo nano /opt/dionaea/etc/dionaea/dionaea.cfg
```

2. Adjust settings like enabled services, logging options, etc.

### 7. Set Up Log Analysis

T-Pot uses Elasticsearch, Logstash, and Kibana (ELK stack) for log analysis.

1. Access Kibana at `https://<your-ip-address>:64296`

2. Create index patterns for T-Pot data

3. Build dashboards to visualize attack data

![Kibana Dashboard](https://example.com/path/to/kibana_dashboard.png)
*Figure 4: Kibana Dashboard for Log Analysis. Replace with an actual screenshot of a Kibana dashboard showing honeypot data.*

### 8. Implement Alerts

Set up alerts in Elasticsearch:

1. In Kibana, go to "Stack Management" > "Alerts and Insights" > "Rules"

2. Create a new rule, e.g., for multiple failed SSH attempts

### 9. Secure the Honeypot

1. Regularly update T-Pot:

```bash
cd /opt/tpot
sudo git pull
sudo ./update.sh
```

2. Monitor system resources to ensure the honeypot doesn't become overwhelmed:

```bash
htop
```

### 10. Analyze Collected Data

1. Use Kibana to analyze attack patterns, most targeted services, etc.

2. Export interesting payloads for further analysis:

```bash
sudo docker cp $(sudo docker ps -qf name=cowrie):/home/cowrie/cowrie-git/var/lib/cowrie/downloads/ /tmp/cowrie_downloads/
```

## Advanced Topics

- Integrating T-Pot with external SIEM systems
- Customizing honeypots to mimic specific services or vulnerabilities
- Implementing machine learning for anomaly detection in honeypot data
- Setting up a distributed honeypot network

## Troubleshooting

- If services fail to start, check Docker logs: `sudo docker logs <container_name>`
- For networking issues, verify iptables rules and Docker network settings
- If facing performance issues, adjust resource allocation in Docker compose files

## Best Practices

1. Isolate your honeypot network from production environments
2. Regularly update and patch your honeypot systems
3. Monitor your honeypots closely to ensure they haven't been compromised
4. Use strong, unique passwords for all honeypot services
5. Analyze and correlate data from multiple honeypots for better insights
6. Be cautious with the data collected, as it may contain malware

## Additional Resources

- [T-Pot GitHub Repository](https://github.com/telekom-security/tpotce)
- [Cowrie Documentation](https://cowrie.readthedocs.io/)
- [Dionaea Documentation](https://dionaea.readthedocs.io/)
- [ELK Stack Documentation](https://www.elastic.co/guide/index.html)
- [Honeypots: Tracking Hackers (Book)](https://www.amazon.com/Honeypots-Tracking-Hackers-Lance-Spitzner/dp/0321108957)

## Glossary

- **Honeypot**: A security mechanism set to detect, deflect, or counteract attempts at unauthorized use of information systems
- **T-Pot**: A multi-honeypot platform based on well-established honeypot daemons
- **Cowrie**: A medium to high interaction SSH and Telnet honeypot
- **Dionaea**: A low-interaction honeypot designed to capture malware
- **ELK Stack**: Elasticsearch, Logstash, and Kibana; used for searching, analyzing, and visualizing log data
- **SIEM**: Security Information and Event Management; provides real-time analysis of security alerts generated by applications and network hardware

