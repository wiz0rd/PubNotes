# Containerlab Network Setup and Troubleshooting

## Index
1. [Initial Setup](#initial-setup)
2. [SNMP Configuration](#snmp-configuration)
3. [SSH Key Management](#ssh-key-management)
4. [Troubleshooting](#troubleshooting)
5. [Making Configurations Persistent](#making-configurations-persistent)
6. [Documentation Links](#documentation-links)

## Initial Setup
- Deployed a spine-leaf topology using Containerlab
- Configured basic networking on CSR, NX-OS, and vJunos devices

## SNMP Configuration

### CSR (IOS-XE) Configuration
```
snmp-server location ContainerlabPop
snmp-server contact WIZ0RD
snmp-server user wiz0rdsw0rld OBSERVIUM v3 auth sha wiz0rdsw0rld priv aes 128 wiz0rdsw0rld
snmp-server group OBSERVIUM v3 priv
snmp-server view all iso included
snmp-server community wiz0rdsw0rld RO
snmp-server host 172.16.40.237 version 3 priv wiz0rdsw0rld
snmp-server enable traps
```

### NX-OS Configuration
```
snmp-server location ContainerlabPop
snmp-server contact WIZ0RD
snmp-server user wiz0rdsw0rld auth sha wiz0rdsw0rld priv aes-128 wiz0rdsw0rld
snmp-server host 172.16.40.237 traps version 3 priv wiz0rdsw0rld
snmp-server enable traps
```

### vJunos Configuration
```
set snmp location ContainerlabPop
set snmp contact WIZ0RD
set snmp v3 usm local-engine user wiz0rdsw0rld authentication-sha authentication-key wiz0rdsw0rld
set snmp v3 usm local-engine user wiz0rdsw0rld privacy-aes128 privacy-key wiz0rdsw0rld
set snmp v3 vacm security-to-group security-model usm security-name wiz0rdsw0rld group OBSERVIUM
set snmp v3 vacm access group OBSERVIUM default-context-prefix security-model usm security-level privacy read-view all
```

## SSH Key Management

### Adding Known Hosts
```bash
for ip in 172.20.20.{2..6}; do
  case $ip in
    172.20.20.2) comment="# csr-spine";;
    172.20.20.3) comment="# csr-leaf";;
    172.20.20.4) comment="# n9kv-spine";;
    172.20.20.5) comment="# n9kv-leaf";;
    172.20.20.6) comment="# vjunosswitch-spine";;
  esac
  ssh-keyscan -H $ip >> ~/.ssh/known_hosts
  echo $comment >> ~/.ssh/known_hosts
done
```

## Troubleshooting

### SNMP Verification
```
snmpwalk -v3 -l authPriv -u wiz0rdsw0rld -a SHA -A wiz0rdsw0rld -x AES -X wiz0rdsw0rld <device_ip> .1.3.6.1.2.1.1.1
```

### SSH Debugging
```
debug ip ssh
```

## Making Configurations Persistent

### CSR (IOS-XE)
```
crypto key generate rsa modulus 2048 exportable
ip ssh rsa keypair-name ssh-key
crypto key export rsa ssh-key pem url flash:ssh-key.pem
crypto key import rsa ssh-key pem url flash:ssh-key.pem
ip ssh rsa keypair-name ssh-key
```

### NX-OS
```
feature ssh
ssh key rsa 2048 force
```

### vJunos
```
set system services ssh root-login allow
set system services ssh authentication-order password
set system services ssh protocol-version v2
request system generate-key-pair ssh-key type rsa size 2048
```

### Containerlab Topology File Modification
```yaml
nodes:
  csr-spine:
    kind: vr-csr
    image: cisco:csr1000v
    binds:
      - ssh-key.pem:/ssh-key.pem
```

## Documentation Links
- [[https://containerlab.dev/|Containerlab Documentation]]
- [[https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/csa/configuration/xe-3s/csa-xe-3s-book/csa-snmp-support.html|Cisco IOS-XE SNMP Configuration]]
- [[https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/system_management/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_System_Management_Configuration_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_System_Management_Configuration_Guide_7x_chapter_010110.html|Cisco NX-OS SNMP Configuration]]
- [[https://www.juniper.net/documentation/us/en/software/junos/network-mgmt/topics/topic-map/snmp-configuring.html|Juniper SNMP Configuration]]
- [[https://www.ssh.com/academy/ssh/keyscan|SSH-Keyscan Documentation]]
- [[https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/security/s1/sec-s1-cr-book/sec-cr-s3.html#wp3436755890|Cisco IOS-XE SSH Configuration]]
- [[https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/security/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_Security_Configuration_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_Security_Configuration_Guide_7x_chapter_01001.html|Cisco NX-OS SSH Configuration]]
- [[https://www.juniper.net/documentation/us/en/software/junos/system-basics/topics/topic-map/ssh-services.html|Juniper SSH Configuration]]
