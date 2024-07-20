# Network Backup Playbook TTP

## Goal
The goal of this TTP is to create an Ansible playbook that automates the process of backing up configurations from network devices, creating a "latest" directory for easy access to the most recent backups, and implementing version control for network configurations.

## Overview
This TTP covers the creation of an Ansible playbook that performs daily backups of network device configurations, organizes them in a structured manner, and maintains a "latest" directory for quick access to the most recent configurations.

![Network Backup Workflow](https://example.com/path/to/network_backup_workflow.png)
*Figure 1: Network Backup Workflow. Replace with an actual diagram showing the backup process flow.*

## Prerequisites
- Ansible installed and configured
- Basic understanding of Ansible playbooks and inventory
- Access to network devices (with appropriate credentials)
- Git installed (for version control)

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

### 2. Create the Ansible Vault

Create an encrypted file to store sensitive data:

```bash
ansible-vault create group_vars/all/vault.yml
```

Add the following content:

```yaml
vault_ansible_password: your_secure_password
```

### 3. Create the Backup Playbook

Create a file named `network_backup.yml`:

```yaml
---
- name: Backup Network Configurations
  hosts: all
  gather_facts: no

  vars:
    backup_path: "/path/to/network_backups"
    latest_path: "{{ backup_path }}/latest"

  tasks:
    - name: Create backup directories
      file:
        path: "{{ item }}"
        state: directory
      loop:
        - "{{ backup_path }}"
        - "{{ latest_path }}"
      run_once: true
      delegate_to: localhost

    - name: Backup device configurations
      ios_config:
        backup: yes
        backup_options:
          filename: "{{ inventory_hostname }}_config.{{ ansible_date_time.date }}.txt"
          dir_path: "{{ backup_path }}/{{ ansible_date_time.date }}"
      register: config_output

    - name: Copy latest configs to latest directory
      copy:
        src: "{{ config_output.backup_path }}"
        dest: "{{ latest_path }}/{{ inventory_hostname }}_config_latest.txt"
      delegate_to: localhost

    - name: Initialize Git repository
      git:
        repo: "{{ backup_path }}"
        version: master
      run_once: true
      delegate_to: localhost

    - name: Commit changes to Git
      shell: |
        cd {{ backup_path }}
        git add .
        git commit -m "Backup taken on {{ ansible_date_time.date }}"
      run_once: true
      delegate_to: localhost
      ignore_errors: yes
```

### 4. Create an Ansible Configuration File

Create `ansible.cfg` in your project directory:

```ini
[defaults]
inventory = network_inventory.yml
vault_password_file = /path/to/vault_password_file
```

### 5. Run the Playbook

Execute the playbook:

```bash
ansible-playbook network_backup.yml
```

### 6. Schedule Daily Backups

Add a cron job to run the playbook daily:

```bash
crontab -e
```

Add the following line:

```
0 1 * * * /usr/bin/ansible-playbook /path/to/network_backup.yml
```

### 7. Verify Backups

Check the backup directory structure:

```
network_backups/
├── 2023-07-16/
│   ├── router1_config.2023-07-16.txt
│   ├── router2_config.2023-07-16.txt
│   ├── switch1_config.2023-07-16.txt
│   └── switch2_config.2023-07-16.txt
├── latest/
│   ├── router1_config_latest.txt
│   ├── router2_config_latest.txt
│   ├── switch1_config_latest.txt
│   └── switch2_config_latest.txt
└── .git/
```

### 8. Implement Backup Rotation

To prevent the backup directory from growing too large, add a task to remove old backups:

```yaml
- name: Remove backups older than 30 days
  find:
    paths: "{{ backup_path }}"
    age: 30d
    recurse: yes
    file_type: directory
  register: old_backups
  run_once: true
  delegate_to: localhost

- name: Delete old backup directories
  file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ old_backups.files }}"
  run_once: true
  delegate_to: localhost
```

### 9. Add Error Handling

Implement error handling to ensure the playbook continues even if a device is unreachable:

```yaml
- name: Backup device configurations
  ios_config:
    backup: yes
    backup_options:
      filename: "{{ inventory_hostname }}_config.{{ ansible_date_time.date }}.txt"
      dir_path: "{{ backup_path }}/{{ ansible_date_time.date }}"
  register: config_output
  ignore_errors: yes

- name: Log backup failures
  lineinfile:
    path: "{{ backup_path }}/backup_failures.log"
    line: "{{ inventory_hostname }} backup failed on {{ ansible_date_time.date }}"
    create: yes
  when: config_output.failed
  delegate_to: localhost
```

### 10. Implement Backup Verification

Add a task to verify the integrity of the backups:

```yaml
- name: Verify backup integrity
  command: "grep '^version' {{ latest_path }}/{{ inventory_hostname }}_config_latest.txt"
  register: verify_result
  failed_when: verify_result.rc != 0
  changed_when: false
  ignore_errors: yes
  delegate_to: localhost

- name: Log verification failures
  lineinfile:
    path: "{{ backup_path }}/verification_failures.log"
    line: "{{ inventory_hostname }} backup verification failed on {{ ansible_date_time.date }}"
    create: yes
  when: verify_result.failed
  delegate_to: localhost
```

## Advanced Topics
- Implementing differential backups to save space
- Encrypting backup files for added security
- Integrating with a centralized backup system
- Implementing automated restore procedures

## Troubleshooting
- Check device connectivity if backups fail
- Verify Ansible has appropriate permissions to write to the backup directory
- Ensure Git is installed and configured properly for version control
- Review logs for any errors during the backup process

## Best Practices
1. Regularly test the restore process from backups
2. Implement monitoring to alert on backup failures
3. Use meaningful commit messages in Git for easy tracking
4. Periodically audit the backup system to ensure it's functioning correctly
5. Implement proper access controls to the backup repository
6. Consider off-site or cloud storage for disaster recovery

## Additional Resources
- [Ansible Network Modules Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [Git Version Control Best Practices](https://www.git-tower.com/learn/git/ebook/en/command-line/appendix/best-practices)
- [Network Configuration Management Best Practices](https://www.ciscopress.com/articles/article.asp?p=2755710&seqNum=5)

## Glossary
- **Playbook**: An Ansible YAML file containing a set of plays to be executed
- **Inventory**: A list of managed nodes in an Ansible environment
- **Vault**: Ansible's built-in encryption for sensitive data
- **Cron**: A time-based job scheduler in Unix-like operating systems
- **Git**: A distributed version control system
- **Differential Backup**: A backup that only contains the data that has changed since the last full backup

