# Ansible Variables TTP

## Goal
The goal of this TTP is to provide a comprehensive understanding of Ansible variables, including their types, scopes, usage in playbooks and roles, and best practices for managing variables in network automation tasks.

## Overview
Variables in Ansible allow you to manage different environments with a single playbook. This TTP covers how to define, use, and manage variables effectively in your network automation projects.

![Ansible Variables Overview](https://example.com/path/to/ansible_variables_overview.png)
*Figure 1: Ansible Variables Overview. Replace with an actual diagram showing different types and scopes of Ansible variables.*

## Prerequisites
- Basic understanding of Ansible (covered in the Ansible Basics TTP)
- Familiarity with YAML syntax
- Basic understanding of Jinja2 templating

## Step-by-step Guide

### 1. Understanding Variable Types

Ansible uses several types of variables:

- String: `hostname: "router1"`
- Number: `vlan_id: 100`
- Boolean: `is_enabled: true`
- List: `vlans: [10, 20, 30]`
- Dictionary: `interface: {name: "GigabitEthernet0/1", ip: "192.168.1.1"}`

### 2. Defining Variables in Playbooks

Create a playbook `network_config.yml`:

```yaml
---
- name: Configure Network Devices
  hosts: all
  vars:
    ntp_server: 10.0.0.1
    syslog_server: 10.0.0.2
  
  tasks:
    - name: Configure NTP
      cisco.ios.ios_ntp:
        server: "{{ ntp_server }}"

    - name: Configure Syslog
      cisco.ios.ios_logging:
        dest: host
        name: "{{ syslog_server }}"
        facility: local7
        level: debugging
```

### 3. Using Variables in Inventory

Edit your inventory file `/etc/ansible/hosts`:

```ini
[routers]
router1 ansible_host=192.168.1.1 location=datacenter1
router2 ansible_host=192.168.1.2 location=datacenter2

[routers:vars]
ansible_network_os=ios
ansible_user=admin
```

### 4. Creating Group and Host Variable Files

Create `group_vars/routers.yml`:

```yaml
---
ntp_server: 10.0.0.1
syslog_server: 10.0.0.2
```

Create `host_vars/router1.yml`:

```yaml
---
local_timezone: EST
```

### 5. Using Variables in Templates

Create a template `templates/interface_config.j2`:

```jinja2
interface {{ interface.name }}
 ip address {{ interface.ip }} {{ interface.subnet_mask }}
 description {{ interface.description | default('Configured by Ansible') }}
```

Use the template in a playbook:

```yaml
---
- name: Configure Interfaces
  hosts: routers
  vars:
    interface:
      name: GigabitEthernet0/1
      ip: 192.168.1.1
      subnet_mask: 255.255.255.0
  
  tasks:
    - name: Apply interface configuration
      cisco.ios.ios_config:
        src: interface_config.j2
```

### 6. Using Variable Files

Create a variable file `vars/network_config.yml`:

```yaml
---
vlans:
  - id: 10
    name: Data
  - id: 20
    name: Voice
  - id: 30
    name: Management
```

Use the variable file in a playbook:

```yaml
---
- name: Configure VLANs
  hosts: switches
  vars_files:
    - vars/network_config.yml
  
  tasks:
    - name: Configure VLANs
      cisco.ios.ios_vlans:
        config:
          - name: "{{ item.name }}"
            vlan_id: "{{ item.id }}"
      loop: "{{ vlans }}"
```

### 7. Using Ansible Facts

Ansible facts are variables that are automatically discovered by Ansible from a managed host. Use them in your playbooks:

```yaml
---
- name: Display Device Information
  hosts: routers
  gather_facts: yes
  
  tasks:
    - name: Show device model
      debug:
        msg: "This device is a {{ ansible_net_model }}"
```

### 8. Variable Precedence

Understand the order of variable precedence (from least to most precedent):

1. Command line values (e.g., `-e "user=my_user"`)
2. role defaults
3. inventory file or script group vars
4. inventory group_vars/all
5. playbook group_vars/all
6. inventory group_vars/*
7. playbook group_vars/*
8. inventory file or script host vars
9. inventory host_vars/*
10. playbook host_vars/*
11. host facts / cached set_facts
12. play vars
13. play vars_prompt
14. play vars_files
15. role vars (defined in role/vars/main.yml)
16. block vars (only for tasks in block)
17. task vars (only for the task)
18. include_vars
19. set_facts / registered vars
20. role (and include_role) params
21. include params
22. extra vars (always win precedence)

### 9. Using set_fact for Dynamic Variables

Use `set_fact` to create variables dynamically:

```yaml
- name: Set interface description
  set_fact:
    interface_description: "{{ inventory_hostname }} - {{ ansible_net_interfaces[interface_name].macaddress }}"

- name: Configure interface description
  cisco.ios.ios_config:
    lines:
      - description {{ interface_description }}
    parents: interface {{ interface_name }}
```

### 10. Using Variable Filters

Ansible provides filters to manipulate variable data:

```yaml
- name: Display uppercase hostname
  debug:
    msg: "Uppercase hostname: {{ inventory_hostname | upper }}"

- name: Get the last octet of IP
  set_fact:
    last_octet: "{{ ansible_host.split('.')[-1] }}"
```

## Advanced Topics
- Using Ansible Vault for encrypting sensitive variables
- Implementing dynamic inventories with variables
- Leveraging custom plugins for complex variable manipulation

## Troubleshooting
- Use `ansible-playbook --list-hosts` to check which hosts a playbook will run against
- Use `ansible-inventory --list` to see all defined variables for your inventory
- Use `debug` module to print variable values during playbook execution

## Best Practices
1. Use meaningful and consistent variable names
2. Leverage group_vars and host_vars for better organization
3. Use defaults in roles for variables that can be overridden
4. Avoid using `vars_prompt` for sensitive data; use Ansible Vault instead
5. Document all variables, especially those intended to be set by users
6. Use `{{ var | default('default_value') }}` for variables that might not be set

## Additional Resources
- [Ansible Variables Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
- [Ansible Variable Precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)
- [Jinja2 Template Designer Documentation](https://jinja.palletsprojects.com/en/2.11.x/templates/)

## Glossary
- **Variable**: A value that can change depending on conditions or information passed to the program
- **Fact**: A special type of variable that is discovered automatically by Ansible from a host
- **Jinja2**: A modern and designer-friendly templating language for Python
- **set_fact**: An Ansible module used to set variables dynamically during playbook execution
- **group_vars**: Variables applied to all hosts in a group
- **host_vars**: Variables applied to specific hosts
- **Ansible Vault**: A feature of Ansible that allows keeping sensitive data such as passwords or keys in encrypted files

