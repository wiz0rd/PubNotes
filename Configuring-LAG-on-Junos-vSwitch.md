# Configuring LACP on Juniper vJunos Switches

## Purpose
This TTP outlines the process for configuring Link Aggregation Control Protocol (LACP) on Juniper vJunos switches in a spine-leaf topology. The goal is to create a functional Link Aggregation Group (LAG) between spine and leaf switches.

## Index
1. [[#Prerequisites]]
2. [[#Spine Switch Configuration]]
3. [[#Leaf Switch Configuration]]
4. [[#Verification]]
5. [[#Troubleshooting]]

## Prerequisites
- Juniper vJunos switches (spine and leaf)
- Access to the switch CLI
- Identified interfaces for the LAG

## Spine Switch Configuration
1. Set the device count for aggregated Ethernet interfaces:
   ```
   [edit]
   user@vjunosswitch-spine# set chassis aggregated-devices ethernet device-count 5
   ```

2. Configure the physical interfaces:
   ```
   [edit]
   user@vjunosswitch-spine# set interfaces ge-0/0/0 ether-options 802.3ad ae0
   user@vjunosswitch-spine# set interfaces ge-0/0/1 ether-options 802.3ad ae0
   ```

3. Configure the ae0 interface:
   ```
   [edit]
   user@vjunosswitch-spine# set interfaces ae0 aggregated-ether-options lacp active
   user@vjunosswitch-spine# set interfaces ae0 unit 0 family ethernet-switching interface-mode trunk
   user@vjunosswitch-spine# set interfaces ae0 unit 0 family ethernet-switching vlan members all
   ```

4. Commit the changes:
   ```
   [edit]
   user@vjunosswitch-spine# commit and-quit
   ```

## Leaf Switch Configuration
1. Repeat steps 1-4 from the Spine Switch Configuration on the leaf switch.

2. Ensure the LACP mode on the leaf switch matches the spine switch (active in this case).

## Verification
1. On both spine and leaf switches, verify the ae0 interface:
   ```
   user@vjunosswitch# show interfaces ae0
   ```

2. Check LACP status:
   ```
   user@vjunosswitch# show lacp interfaces
   ```

3. Confirm that:
   - The ae0 interface is enabled and the physical link is up
   - The speed shows the sum of all member link speeds
   - LACP is active on both actor and partner
   - Member links are in "Collecting distributing" state

## Troubleshooting
- If interfaces show as down, check physical connections
- Ensure configurations match on both spine and leaf switches
- Verify that the correct family (ethernet-switching) is configured on ae0
- Check system logs for any error messages related to LACP or interface configuration

## Related Topics
- [[Link Aggregation]]
- [[LACP]]
- [[Juniper vJunos Configuration]]
- [[Spine-Leaf Topology]]

## External Resources
- [Juniper Documentation on Configuring LACP](https://www.juniper.net/documentation/us/en/software/junos/interfaces-ethernet/topics/topic-map/aggregated-ethernet-interfaces-lacp.html)
- [IEEE 802.3ad Standard](https://standards.ieee.org/standard/802_3ad-2000.html)
