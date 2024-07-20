# Device Onboarding Playbook TTP

## Goal
The goal of this TTP is to create an Ansible playbook that automates the process of onboarding new network devices. This includes initial configuration, setting up TACACS+ and SNMP, and standardizing the device setup across your network infrastructure.

## Overview
This TTP covers the creation of an Ansible playbook that handles the initial configuration and standardization of new network devices, ensuring consistency and efficiency in the onboarding process.

![Device Onboarding Workflow](https://example.com/path/to/device_onboarding_workflow.png)
*Figure 1: Device Onboarding Workflow. Replace with an actual diagram showing the steps of the onboarding process.*

## Prerequisites
- Ansible installed and configured
- Basic understanding of network device configuration
- Access to network devices (with appropriate credentials)
- TACACS+ and SNMP servers set up in your environment

## Step-by-step Guide

### 1. Create the Inventory

Create an inventory file `new_devices.yml`:

```yaml
all:
  children:
    new_devices:
      hosts:
        new_router:
          ansible_host: 192.168.1.10
        new_switch:
          ansible_host: 192.168.1.20
  vars:
    ansible_network_os: ios
    ansible_connection: network_cli
    ansible_user: setup_user
    ansible_password: "{{ vault_setup_password }}"
```

### 2. Create Variable Files

Create a `group_vars` directory and add a file for your new devices:

```bash
mkdir -p group_vars/new_devices
touch group_vars/new_devices/vars.yml
```

Add the following content to `vars.yml`:

```yaml
---
hostname_prefix: ACME-
domain_name: acme.com
ntp_servers:
  - 10.0.0.1
  - 10.0.0.2
dns_servers:
  - 8.8.8.8
  - 8.8.4.4
tacacs_servers:
  - 10.1.1.1
  - 10.1.1.2
snmp_community: "{{ vault_snmp_community }}"
snmp_location: "Data Center 1"
snmp_contact: "noc@acme.com"
```

### 3. Create an Ansible Vault

Create an encrypted file to store sensitive data:

```bash
ansible-vault create group_vars/new_devices/vault.yml
```

Add the following content:

```yaml
vault_setup_password: initial_setup_password
vault_enable_secret: secure_enable_password
vault_tacacs_key: secure_tacacs_key
vault_snmp_community: secure_snmp_community
```

### 4. Create the Onboarding Playbook

Create a file named `onboard_device.yml`:

```yaml
---
- name: Onboard New Network Devices
  hosts: new_devices
  gather_facts: no

  tasks:
    - name: Set hostname
      ios_config:
        lines:
          - hostname {{ hostname_prefix }}{{ inventory_hostname }}

    - name: Configure domain name
      ios_config:
        lines:
          - ip domain-name {{ domain_name }}

    - name: Configure NTP servers
      ios_config:
        lines:
          - ntp server {{ item }}
      loop: "{{ ntp_servers }}"

    - name: Configure DNS servers
      ios_config:
        lines:
          - ip name-server {{ item }}
      loop: "{{ dns_servers }}"

    - name: Configure TACACS+
      ios_config:
        lines:
          - tacacs-server host {{ item }}
          - tacacs-server key {{ vault_tacacs_key }}
      loop: "{{ tacacs_servers }}"

    - name: Configure SNMP
      ios_config:
        lines:
          - snmp-server community {{ vault_snmp_community }} RO
          - snmp-server location {{ snmp_location }}
          - snmp-server contact {{ snmp_contact }}

    - name: Configure enable secret
      ios_config:
        lines:
          - enable secret {{ vault_enable_secret }}

    - name: Configure SSH
      ios_config:
        lines:
          - ip ssh version 2
          - line vty 0 4
          - transport input ssh

    - name: Save configuration
      ios_command:
        commands: write memory

    - name: Verify configuration
      ios_command:
        commands:
          - show run | include hostname
          - show run | include domain-name
          - show run | include ntp
          - show run | include name-server
          - show run | include tacacs
          - show run | include snmp-server
      register: verify_output

    - name: Display verification results
      debug:
        var: verify_output.stdout_lines
```

### 5. Create a Pre-onboarding Checklist

Create a pre-onboarding checklist playbook `pre_onboarding_checklist.yml`:

```yaml
---
- name: Pre-onboarding Checklist
  hosts: new_devices
  gather_facts: no

  tasks:
    - name: Check device reachability
      ios_command:
        commands: show version
      register: reachability_check
      ignore_errors: yes

    - name: Verify initial access
      assert:
        that:
          - reachability_check is success
        fail_msg: "Unable to reach {{ inventory_hostname }}"

    - name: Check available flash memory
      ios_command:
        commands: show flash: | include bytes free
      register: flash_check

    - name: Verify sufficient flash memory
      assert:
        that:
          - flash_check.stdout[0] | regex_search('([0-9]+) bytes free') | regex_replace('([0-9]+) bytes free', '\\1') | int > 50000000
        fail_msg: "Insufficient flash memory on {{ inventory_hostname }}"

    - name: Check current configuration
      ios_command:
        commands: show run
      register: config_check

    - name: Verify device is in factory default state
      assert:
        that:
          - "'hostname' not in config_check.stdout[0]"
        fail_msg: "{{ inventory_hostname }} may not be in factory default state"
```

### 6. Create a Post-onboarding Verification Playbook

Create a post-onboarding verification playbook `post_onboarding_verify.yml`:

```yaml
---
- name: Post-onboarding Verification
  hosts: new_devices
  gather_facts: no

  tasks:
    - name: Verify hostname
      ios_command:
        commands: show run | include hostname
      register: hostname_check
      failed_when: "hostname_prefix not in hostname_check.stdout[0]"

    - name: Verify TACACS+
      ios_command:
        commands: show run | include tacacs
      register: tacacs_check
      failed_when: "tacacs_servers | length != tacacs_check.stdout_lines | length"

    - name: Verify SNMP
      ios_command:
        commands: show run | include snmp-server
      register: snmp_check
      failed_when: "snmp_community not in snmp_check.stdout[0] or snmp_location not in snmp_check.stdout[0] or snmp_contact not in snmp_check.stdout[0]"

    - name: Verify SSH
      ios_command:
        commands: show ip ssh
      register: ssh_check
      failed_when: "'SSH Enabled - version 2.0' not in ssh_check.stdout[0]"

    - name: Generate onboarding report
      template:
        src: templates/onboarding_report.j2
        dest: "reports/{{ inventory_hostname }}_onboarding_report.txt"
      delegate_to: localhost
```

### 7. Create a Main Onboarding Playbook

Create a main playbook `main_onboarding.yml` that ties everything together:

```yaml
---
- name: Main Onboarding Playbook
  hosts: new_devices
  gather_facts: no

  tasks:
    - name: Run pre-onboarding checklist
      import_playbook: pre_onboarding_checklist.yml

    - name: Perform device onboarding
      import_playbook: onboard_device.yml

    - name: Run post-onboarding verification
      import_playbook: post_onboarding_verify.yml
```

### 8. Run the Onboarding Process

Execute the main onboarding playbook:

```bash
ansible-playbook -i new_devices.yml main_onboarding.yml --ask-vault-pass
```

### 9. Implement Error Handling and Rollback

Add error handling and rollback capabilities to the `onboard_device.yml` playbook:

```yaml
  tasks:
    # ... previous tasks ...

    - name: Save configuration
      ios_command:
        commands: write memory
      register: save_result

    - name: Implement rollback on failure
      block:
        - name: Verify critical services
          ios_command:
            commands:
              - show ip ssh
              - show tacacs
          register: critical_services_check
          failed_when: "'SSH Enabled' not in critical_services_check.stdout[0] or 'TACACS' not in critical_services_check.stdout[1]"
      rescue:
        - name: Rollback configuration
          ios_config:
            rollback: 1
          when: save_result is success

        - name: Notify of rollback
          debug:
            msg: "Configuration rollback performed due to critical service failure"
```

### 10. Document the Onboarding Process

Create a README.md file in your project directory with instructions on how to use the onboarding playbooks, including prerequisites, execution steps, and troubleshooting tips.

## Advanced Topics
- Integrating with a IPAM (IP Address Management) system for automatic IP assignment
- Implementing device-specific configurations based on hardware model
- Creating a self-service portal for network operators to initiate the onboarding process
- Integrating with a CMDB (Configuration Management Database) to update device inventory

## Troubleshooting
- Verify network connectivity to the new devices
- Check that the provided credentials are correct
- Ensure that the necessary ports (SSH, SNMP) are open on the network
- Review Ansible logs for detailed error messages
- Use `--check` mode to test playbooks without making changes

## Best Practices
1. Always run the pre-onboarding checklist before proceeding with configuration
2. Use version control for all playbooks and templates
3. Implement proper error handling and rollback mechanisms
4. Regularly update and test the onboarding playbooks
5. Use Ansible Vault to secure sensitive information
6. Document any device-specific exceptions or special configurations
7. Implement a peer review process for onboarding playbook changes

## Additional Resources
- [Ansible Network Automation Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [Cisco IOS Configuration Guide](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-15-4m-t/products-installation-and-configuration-guides-list.html)
- [Network Device Onboarding Best Practices](https://www.ciscopress.com/articles/article.asp?p=2999406&seqNum=4)

## Glossary
- **TACACS+**: Terminal Access Controller Access-Control System Plus, a protocol providing centralized authentication for network devices
- **SNMP**: Simple Network Management Protocol, used for collecting information from, and configuring, network devices
- **Onboarding**: The process of adding a new device to a network and configuring it for operation
- **Rollback**: The process of reverting to a previous configuration state
- **IPAM**: IP Address Management, a means of planning, tracking, and managing IP address space in a network
- **CMDB**: Configuration Management Database, a repository that acts as a data warehouse for IT installations

