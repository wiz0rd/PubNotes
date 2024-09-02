# TTP: Allowing External VM's Access to Containerlab Devices for Services like SNMPv3

## Overview

This procedure outlines the steps to configure a Containerlab host and external VMs to allow access to Containerlab devices for services such as SNMPv3. This setup enables network management and monitoring tools to interact with virtualized network devices.

## Prerequisites

- A host machine running Containerlab (referred to as "Containerlab host")
- External VM(s) that need access to Containerlab devices
- Root or sudo access on both the Containerlab host and external VMs
- A defined Containerlab topology with management IP addresses

## Dependencies on Containerlab Topology

Before proceeding with this TTP, ensure that your Containerlab topology is properly defined with management IP addresses for each device. The management network subnet defined in your topology file is crucial for setting up the correct NAT and routing rules.

Example snippet from a Containerlab topology file (`topology.yml`):

```yaml
name: my-topology
mgmt:
  network: custom_mgmt
  ipv4-subnet: 172.20.20.0/24

topology:
  nodes:
    router1:
      kind: nokia_sr
      mgmt-ipv4: 172.20.20.2
    
    switch1:
      kind: arista_ceos
      mgmt-ipv4: 172.20.20.3
```

In this example, the management subnet is 172.20.20.0/24. You'll need to use this subnet in the NAT and routing configurations that follow. Adjust the IP addresses and subnets in this TTP to match your specific Containerlab topology.

## Procedure

### 1. Containerlab Host Configuration

#### 1.1 Enable IP Forwarding

1. Open the sysctl configuration file:
   ```bash
   sudo nano /etc/sysctl.conf
   ```

2. Add or uncomment the following line:
   ```
   net.ipv4.ip_forward=1
   ```

3. Save and exit the editor.

4. Apply the changes:
   ```bash
   sudo sysctl -p
   ```

#### 1.2 Configure NAT and Forwarding Rules

1. Add the following iptables rules:
   ```bash
   sudo iptables -t nat -A POSTROUTING -s 172.20.20.0/24 -o enxa0cec889efea -j MASQUERADE
   sudo iptables -A FORWARD -i enxa0cec889efea -o br-e3af8a0fe6e1 -j ACCEPT
   sudo iptables -A FORWARD -i br-e3af8a0fe6e1 -o enxa0cec889efea -j ACCEPT
   ```
   
   Note: Replace `172.20.20.0/24` with your Containerlab management subnet, `enxa0cec889efea` with your external interface name, and `br-e3af8a0fe6e1` with your Containerlab bridge interface name.

2. Install iptables-persistent to make rules permanent:
   ```bash
   sudo apt-get install iptables-persistent
   sudo netfilter-persistent save
   ```

#### 1.3 Create a Systemd Service for NAT Rules

1. Create a new systemd service file:
   ```bash
   sudo nano /etc/systemd/system/containerlab-nat.service
   ```

2. Add the following content:
   ```
   [Unit]
   Description=Containerlab NAT rules
   After=network.target

   [Service]
   Type=oneshot
   ExecStart=/sbin/iptables-restore /etc/iptables/rules.v4
   ExecStart=/sbin/iptables -t nat -A POSTROUTING -s 172.20.20.0/24 -o enxa0cec889efea -j MASQUERADE
   ExecStart=/sbin/iptables -A FORWARD -i enxa0cec889efea -o br-e3af8a0fe6e1 -j ACCEPT
   ExecStart=/sbin/iptables -A FORWARD -i br-e3af8a0fe6e1 -o enxa0cec889efea -j ACCEPT
   RemainAfterExit=yes

   [Install]
   WantedBy=multi-user.target
   ```

   Note: Adjust the subnet and interface names as in step 1.2 to match your Containerlab topology.

3. Save and exit the editor.

4. Enable the service:
   ```bash
   sudo systemctl enable containerlab-nat.service
   ```

### 2. External VM Configuration

#### 2.1 Add Static Route

Choose one of the following methods based on your network configuration system:

##### Option A: For systems using /etc/network/interfaces

1. Edit the interfaces file:
   ```bash
   sudo nano /etc/network/interfaces
   ```

2. Add the following line at the end of the file or in the appropriate interface section:
   ```
   up ip route add 172.20.20.0/24 via 172.16.40.5
   ```

   Note: Replace `172.20.20.0/24` with your Containerlab management subnet and `172.16.40.5` with the IP of your Containerlab host.

3. Save and exit the editor.

##### Option B: For systems using Netplan

1. Edit the appropriate Netplan configuration file:
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

   Note: The filename might be different on your system.

2. Add the route to the configuration:
   ```yaml
   network:
     version: 2
     ethernets:
       ens160:  # Replace with your actual interface name
         routes:
           - to: 172.20.20.0/24
             via: 172.16.40.5
   ```

   Note: Adjust the subnet and via IP as in Option A to match your Containerlab topology and host IP.

3. Save and exit the editor.

4. Apply the changes:
   ```bash
   sudo netplan apply
   ```

## Verification

1. From the external VM, ping a Containerlab device using its management IP (e.g., 172.20.20.2 or 172.20.20.3 from the example topology):
   ```bash
   ping 172.20.20.2
   ```

2. If the ping is successful, try accessing the required service (e.g., SNMPv3) from the external VM to the Containerlab device.

## Troubleshooting

- If connectivity fails, check the routing table on both the Containerlab host and external VM using `ip route`.
- Verify iptables rules on the Containerlab host with `sudo iptables -L -v -n`.
- Check for any firewall rules that might be blocking traffic.
- Ensure that the management IP addresses in your Containerlab topology file match the subnet you're using in the NAT and routing configurations.

## Notes

- Always refer to your Containerlab topology file for the correct management subnet and IP addresses when configuring NAT and routing rules.
- Ensure that the Containerlab devices are configured to allow access for the required services (e.g., SNMPv3 configuration on the virtual network devices).
- If you modify your Containerlab topology, particularly the management network configuration, remember to update your NAT and routing rules accordingly.

## Related Documentation

- [[Containerlab Documentation]]
- [[iptables Manual]]
- [[Netplan Configuration]]
- [[Ubuntu Networking Guide]]

## External Links

- [Containerlab Official Documentation](https://containerlab.srlinux.dev/)
- [iptables Documentation](https://netfilter.org/documentation/)
- [Netplan Documentation](https://netplan.io/documentation)
- [Ubuntu Networking Guide](https://ubuntu.com/server/docs/network-configuration)

