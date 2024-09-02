# Network Device Setup and Backup TTP

## Index

1. [[Requirements]]
2. [[Juniper Public Key Configuration]]
3. [[Cisco CSR Public Key Configuration]]
4. [[NX-OS Public Key Configuration]]
5. [[Ansible Inventory Configuration]]
6. [[Ansible Backup Playbook]]
7. [[Links]]

## Requirements

Before starting, ensure you have the following packages installed:

```bash
pip install ansible ansible-pylibssh paramiko ncclient
ansible-galaxy collection install cisco.ios cisco.nxos junipernetworks.junos
```
Also make sure you add all of the keys to your known hosts
```
ssh-keyscan -H 172.20.20.2 172.20.20.3 172.20.20.4 172.20.20.5 172.20.20.6 172.20.20.7 >> ~/.ssh/known_hosts
```

## Juniper Public Key Configuration

1. SSH into the Juniper device
2. Enter configuration mode:
   ```
   configure
   ```
3. Add the SSH public key for the admin user:
   ```
   set system login user admin authentication ssh-rsa "YOUR_PUBLIC_KEY_HERE"
   ```
4. Commit the changes:
   ```
   commit
   ```
5. Exit configuration mode:
   ```
   exit
   ```

## Cisco CSR Public Key Configuration

1. SSH into the CSR device
2. Enter configuration mode:
   ```
   configure terminal
   ```
3. Configure SSH to use public key authentication:
   ```
   ip ssh pubkey-chain
   ```
4. Enter the username configuration mode:
   ```
   username admin
   ```
5. Add the SSH public key (you may need to break it into multiple lines):
   ```
   key-string
   AAAAB3NzaC1yc2EAA[...]
   [...]
   exit
   ```
6. Exit the SSH public key configuration:
   ```
   exit
   ```
7. Save the configuration:
   ```
   end
   write memory
   ```
8. Update your local SSH config file (`~/.ssh/config`):
   ```
   Host csr-spine
       hostname=172.20.20.2
       User admin
       IdentityFile ~/.ssh/id_rsa
   Host csr-leaf
       hostname=172.20.20.3
       User admin
       IdentityFile ~/.ssh/id_rsa
   ```

## NX-OS Public Key Configuration

1. SSH into the NX-OS device
2. Enter configuration mode:
   ```
   configure terminal
   ```
3. Add the SSH key for the admin user:
   ```
   username admin sshkey ssh-rsa AAAAB3NzaC1yc2EAAA... [full key] ...dn user@host
   ```
4. Exit configuration mode:
   ```
   end
   ```
5. Save the configuration:
   ```
   copy running-config startup-config
   ```

## Ansible Inventory Configuration

Create or update your Ansible inventory file (`newInv.yml`) with the following content:

```yaml
all:
  vars:
    ansible_httpapi_use_proxy: false
  children:
    juniper_vjunosswitch:
      vars:
        ansible_connection: netconf
        ansible_network_os: junos
        ansible_user: admin
        ansible_password: admin@123
      hosts:
        clab-spine-leaf-topology-vjunosswitch-leaf:
          ansible_host: 172.20.20.7
        clab-spine-leaf-topology-vjunosswitch-spine:
          ansible_host: 172.20.20.6
    vr-csr:
      vars:
        ansible_connection: network_cli
        ansible_network_os: ios
        ansible_user: admin
        ansible_password: admin
      hosts:
        clab-spine-leaf-topology-csr-leaf:
          ansible_host: 172.20.20.3
        clab-spine-leaf-topology-csr-spine:
          ansible_host: 172.20.20.2
    vr-n9kv:
      vars:
        ansible_connection: network_cli
        ansible_network_os: nxos
        ansible_user: admin
        ansible_password: admin
      hosts:
        clab-spine-leaf-topology-n9kv-leaf:
          ansible_host: 172.20.20.5
        clab-spine-leaf-topology-n9kv-spine:
          ansible_host: 172.20.20.4
```

## Ansible Backup Playbook

Create an Ansible playbook (`backupConfigs.yml`) with the following content:

```yaml
---
- name: Backup Network Device Configurations
  hosts: all
  gather_facts: no
  vars:
    backup_dir: "/home/wiz0rd/Projects/git/Containerlab/Labs/ansible-6/clab-spine-leaf-topology"
    ansible_user: admin
    ansible_ssh_private_key_file: "~/.ssh/id_rsa"
  tasks:
    - name: Backup Juniper vJunos configurations
      junipernetworks.junos.junos_command:
        commands: 
          - show configuration
      register: juniper_config
      when: "'juniper_vjunosswitch' in group_names"
      vars:
        ansible_connection: netconf
        ansible_network_os: junos

    - name: Save Juniper configuration to file
      copy:
        content: "{{ juniper_config.stdout[0] }}"
        dest: "{{ backup_dir }}/{{ inventory_hostname | regex_replace('clab-spine-leaf-topology-', '') }}/config/backup.conf"
      when: "'juniper_vjunosswitch' in group_names"

    - name: Backup Cisco CSR configurations
      cisco.ios.ios_config:
        backup: yes
        backup_options:
          filename: "backup.conf"
          dir_path: "{{ backup_dir }}/{{ inventory_hostname | regex_replace('clab-spine-leaf-topology-', '') }}/config"
      register: cisco_csr_backup
      when: "'vr-csr' in group_names"
      vars:
        ansible_connection: network_cli
        ansible_network_os: ios

    - name: Backup Cisco NX-OS configurations
      cisco.nxos.nxos_config:
        backup: yes
        backup_options:
          filename: "backup.conf"
          dir_path: "{{ backup_dir }}/{{ inventory_hostname | regex_replace('clab-spine-leaf-topology-', '') }}/config"
      register: cisco_nxos_backup
      when: "'vr-n9kv' in group_names"
      vars:
        ansible_connection: network_cli
        ansible_network_os: nxos

    - name: Display backup status
      debug:
        msg: 
          - "Configuration for {{ inventory_hostname }} has been backed up to {{ backup_dir }}/{{ inventory_hostname | regex_replace('clab-spine-leaf-topology-', '') }}/config/backup.conf"
      when: juniper_config.changed or cisco_csr_backup.changed or cisco_nxos_backup.changed

```

To run the playbook:

```bash
ansible-playbook -i newInv.yml backupConfigs.yml
```

## Links

- [Juniper Configuration Guide](https://www.juniper.net/documentation/us/en/software/junos/user-access/topics/topic-map/user-access-authentication-configuration.html)
- [Cisco IOS SSH Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_usr_ssh/configuration/xe-16/sec-usr-ssh-xe-16-book/sec-usr-ssh-sec-shell.html)
- [Cisco NX-OS SSH Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/6-x/security/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_Security_Configuration_Guide/b_Cisco_Nexus_9000_Series_NX-OS_Security_Configuration_Guide_chapter_01010.html)
- [Ansible Network Automation Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [ContainerLab Documentation](https://containerlab.srlinux.dev/)
