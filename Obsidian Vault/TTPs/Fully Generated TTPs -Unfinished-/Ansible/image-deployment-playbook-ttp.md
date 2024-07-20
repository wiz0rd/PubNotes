# Image Deployment Playbook TTP

## Goal
The goal of this TTP is to create an Ansible playbook that automates the process of deploying new images to network devices. This includes verifying current versions, transferring new images, upgrading devices, and implementing rollback procedures if necessary.

## Overview
This TTP covers the creation of an Ansible playbook that manages the entire lifecycle of network device image upgrades, ensuring a consistent and reliable process across your infrastructure.

![Image Deployment Workflow](https://example.com/path/to/image_deployment_workflow.png)
*Figure 1: Image Deployment Workflow. Replace with an actual diagram showing the steps of the image deployment process.*

## Prerequisites
- Ansible installed and configured
- Access to network devices with appropriate credentials
- SFTP server set up with the new images
- Basic understanding of network device image upgrade processes

## Step-by-step Guide

### 1. Create the Inventory

Create an inventory file `network_inventory.yml`:

```yaml
all:
  children:
    routers:
      hosts:
        router1:
          ansible_host: 192.168.1.1
        router2:
          ansible_host: 192.168.1.2
    switches:
      hosts:
        switch1:
          ansible_host: 192.168.1.101
        switch2:
          ansible_host: 192.168.1.102
  vars:
    ansible_network_os: ios
    ansible_connection: network_cli
    ansible_user: admin
    ansible_password: "{{ vault_ansible_password }}"
```

### 2. Create Variable Files

Create a `group_vars` directory and add a file for your devices:

```bash
mkdir -p group_vars/all
touch group_vars/all/vars.yml
```

Add the following content to `vars.yml`:

```yaml
---
sftp_server: 10.0.0.5
sftp_user: sftp_user
sftp_password: "{{ vault_sftp_password }}"
image_directory: /images
new_image_version: 15.2(4)M1
new_image_filename: c2900-universalk9-mz.SPA.152-4.M1.bin
```

### 3. Create an Ansible Vault

Create an encrypted file to store sensitive data:

```bash
ansible-vault create group_vars/all/vault.yml
```

Add the following content:

```yaml
vault_ansible_password: secure_ansible_password
vault_sftp_password: secure_sftp_password
```

### 4. Create the Image Deployment Playbook

Create a file named `deploy_image.yml`:

```yaml
---
- name: Deploy New Image to Network Devices
  hosts: all
  gather_facts: no

  tasks:
    - name: Check current image version
      ios_command:
        commands: show version | include Version
      register: current_version

    - name: Determine if upgrade is needed
      set_fact:
        upgrade_needed: "{{ new_image_version not in current_version.stdout[0] }}"

    - name: Check available flash space
      ios_command:
        commands: show flash: | include bytes free
      register: flash_space
      when: upgrade_needed

    - name: Verify sufficient space for new image
      assert:
        that:
          - flash_space.stdout[0] | regex_search('([0-9]+) bytes free') | regex_replace('([0-9]+) bytes free', '\\1') | int > 100000000
        fail_msg: "Insufficient space in flash for new image on {{ inventory_hostname }}"
      when: upgrade_needed

    - name: Transfer new image via SFTP
      net_put:
        src: "{{ image_directory }}/{{ new_image_filename }}"
        dest: "flash:/{{ new_image_filename }}"
        protocol: sftp
        username: "{{ sftp_user }}"
        password: "{{ sftp_password }}"
      when: upgrade_needed

    - name: Verify image integrity
      ios_command:
        commands: "verify /md5 flash:/{{ new_image_filename }}"
      register: md5_result
      when: upgrade_needed

    - name: Set boot image
      ios_config:
        lines:
          - "no boot system"
          - "boot system flash:/{{ new_image_filename }}"
      when: upgrade_needed

    - name: Save running config
      ios_command:
        commands: write memory
      when: upgrade_needed

    - name: Reload device
      ios_command:
        commands:
          - command: "reload"
            prompt: "Proceed with reload"
            answer: "yes"
      when: upgrade_needed

    - name: Wait for device to come back online
      wait_for:
        host: "{{ ansible_host }}"
        port: 22
        delay: 60
        timeout: 300
      when: upgrade_needed
      delegate_to: localhost

    - name: Verify new image version
      ios_command:
        commands: show version | include Version
      register: new_version
      when: upgrade_needed

    - name: Confirm successful upgrade
      assert:
        that:
          - new_image_version in new_version.stdout[0]
        fail_msg: "Upgrade failed on {{ inventory_hostname }}"
      when: upgrade_needed

    - name: Remove old image file
      ios_command:
        commands: "delete /force flash:/{{ item }}"
      loop: "{{ old_image_files }}"
      when: upgrade_needed and remove_old_images
```

### 5. Create a Pre-deployment Checklist

Create a pre-deployment checklist playbook `pre_deployment_checklist.yml`:

```yaml
---
- name: Pre-deployment Checklist
  hosts: all
  gather_facts: no

  tasks:
    - name: Check device reachability
      ios_command:
        commands: show version
      register: reachability_check
      ignore_errors: yes

    - name: Verify device reachability
      assert:
        that:
          - reachability_check is success
        fail_msg: "Unable to reach {{ inventory_hostname }}"

    - name: Check current configuration
      ios_command:
        commands: show run
      register: config_check

    - name: Backup current configuration
      copy:
        content: "{{ config_check.stdout[0] }}"
        dest: "backups/{{ inventory_hostname }}_pre_upgrade_config.txt"
      delegate_to: localhost

    - name: Verify SFTP server reachability
      net_ping:
        dest: "{{ sftp_server }}"
      register: sftp_reachability

    - name: Confirm SFTP server is reachable
      assert:
        that:
          - sftp_reachability is success
        fail_msg: "SFTP server {{ sftp_server }} is not reachable"
```

### 6. Create a Post-deployment Verification Playbook

Create a post-deployment verification playbook `post_deployment_verify.yml`:

```yaml
---
- name: Post-deployment Verification
  hosts: all
  gather_facts: no

  tasks:
    - name: Verify new image version
      ios_command:
        commands: show version | include Version
      register: version_check
      failed_when: new_image_version not in version_check.stdout[0]

    - name: Check critical services
      ios_command:
        commands:
          - show ip interface brief
          - show ip route summary
          - show ip ospf neighbor
      register: service_check

    - name: Verify critical services are operational
      assert:
        that:
          - "'up' in service_check.stdout[0]"
          - "'Total number of prefixes' in service_check.stdout[1]"
          - "'FULL' in service_check.stdout[2]"
        fail_msg: "Critical services check failed on {{ inventory_hostname }}"

    - name: Generate upgrade report
      template:
        src: templates/upgrade_report.j2
        dest: "reports/{{ inventory_hostname }}_upgrade_report.txt"
      delegate_to: localhost
```

### 7. Create a Main Deployment Playbook

Create a main playbook `main_deployment.yml` that ties everything together:

```yaml
---
- name: Main Image Deployment Playbook
  hosts: all
  gather_facts: no

  tasks:
    - name: Run pre-deployment checklist
      import_playbook: pre_deployment_checklist.yml

    - name: Perform image deployment
      import_playbook: deploy_image.yml

    - name: Run post-deployment verification
      import_playbook: post_deployment_verify.yml
```

### 8. Run the Deployment Process

Execute the main deployment playbook:

```bash
ansible-playbook -i network_inventory.yml main_deployment.yml --ask-vault-pass
```

### 9. Implement Rollback Procedure

Add a rollback task to the `deploy_image.yml` playbook:

```yaml
    - name: Rollback on failure
      block:
        - name: Revert to previous image
          ios_config:
            lines:
              - "no boot system"
              - "boot system flash:/{{ previous_image_filename }}"
          when: upgrade_needed and (new_image_version not in new_version.stdout[0])

        - name: Reload device for rollback
          ios_command:
            commands:
              - command: "reload"
                prompt: "Proceed with reload"
                answer: "yes"
          when: upgrade_needed and (new_image_version not in new_version.stdout[0])

        - name: Wait for device to come back online after rollback
          wait_for:
            host: "{{ ansible_host }}"
            port: 22
            delay: 60
            timeout: 300
          when: upgrade_needed and (new_image_version not in new_version.stdout[0])
          delegate_to: localhost

      rescue:
        - name: Notify of rollback failure
          debug:
            msg: "CRITICAL: Rollback failed on {{ inventory_hostname }}"
```

### 10. Document the Deployment Process

Create a README.md file in your project directory with instructions on how to use the deployment playbooks, including prerequisites, execution steps, and troubleshooting tips.

## Advanced Topics
- Implementing a phased rollout strategy for large networks
- Creating a self-service portal for network operators to initiate upgrades
- Integrating with a change management system
- Implementing automated testing of network functionality post-upgrade

## Troubleshooting
- Verify network connectivity to the devices and SFTP server
- Check that the provided credentials are correct
- Ensure sufficient space is available on the devices for the new image
- Review Ansible logs for detailed error messages
- Use `--check` mode to test playbooks without making changes

## Best Practices
1. Always run the pre-deployment checklist before proceeding with the upgrade
2. Use version control for all playbooks and templates
3. Implement proper error handling and rollback mechanisms
4. Test the upgrade process in a lab environment before applying to production
5. Use Ansible Vault to secure sensitive information
6. Keep a backup of the current image and configuration before upgrading
7. Implement a peer review process for deployment playbook changes

## Additional Resources
- [Ansible Network Automation Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [Cisco IOS Upgrade Guide](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-software-releases-122-mainline/19582-swupgrade.html)
- [Network Device Upgrade Best Practices](https://www.ciscopress.com/articles/article.asp?p=2999406&seqNum=5)

## Glossary
- **SFTP**: Secure File Transfer Protocol, a network protocol for secure file transfer
- **MD5**: Message Digest algorithm 5, used to verify file integrity
- **Rollback**: The process of reverting to a previous software version or configuration state
- **Flash**: Non-volatile storage used in network devices to store system images and configuration files
- **Boot System**: A configuration command used to specify the system image to load during device startup
- **Upgrade**: The process of installing a newer version of software or firmware on a device

