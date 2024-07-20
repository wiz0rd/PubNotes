# Ansible Roles TTP

## Goal
The goal of this TTP is to provide a comprehensive understanding of Ansible roles, including how to create, organize, and effectively use roles in network automation tasks.

## Overview
Ansible roles provide a way to group multiple tasks together and encapsulate data needed to accomplish those tasks. This TTP covers the structure of roles, how to create and use them, and best practices for role development in network automation contexts.

![Ansible Roles Structure](https://example.com/path/to/ansible_roles_structure.png)
*Figure 1: Ansible Roles Structure. Replace with an actual diagram showing the directory structure and components of an Ansible role.*

## Prerequisites
- Basic understanding of Ansible (covered in the Ansible Basics TTP)
- Familiarity with YAML syntax
- Basic understanding of network configurations

## Step-by-step Guide

### 1. Understand Role Structure

An Ansible role has a defined directory structure:

```
rolename/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
└── vars/
    └── main.yml
```

### 2. Create a Basic Role

Create a new role for configuring NTP on network devices:

```bash
ansible-galaxy init configure_ntp
cd configure_ntp
```

### 3. Define Tasks

Edit `tasks/main.yml`:

```yaml
---
- name: Ensure NTP is configured
  cisco.ios.ios_ntp:
    server: "{{ ntp_server }}"
    source_int: "{{ ntp_source_interface }}"
  notify: Restart NTP

- name: Enable NTP
  cisco.ios.ios_config:
    lines:
      - ntp enable
```

### 4. Set Default Variables

Edit `defaults/main.yml`:

```yaml
---
ntp_server: 10.0.0.1
ntp_source_interface: Loopback0
```

### 5. Create Handlers

Edit `handlers/main.yml`:

```yaml
---
- name: Restart NTP
  cisco.ios.ios_config:
    lines:
      - no ntp enable
      - ntp enable
```

### 6. Use the Role in a Playbook

Create a new playbook `configure_network.yml`:

```yaml
---
- name: Configure Network Devices
  hosts: network
  gather_facts: no

  roles:
    - configure_ntp
```

### 7. Override Role Variables

You can override default variables in your playbook:

```yaml
---
- name: Configure Network Devices
  hosts: network
  gather_facts: no

  roles:
    - role: configure_ntp
      vars:
        ntp_server: 192.168.1.100
```

### 8. Create a Role with Templates

For more complex configurations, use templates. Create `templates/ospf.j2`:

```jinja2
router ospf {{ ospf_process_id }}
 router-id {{ ospf_router_id }}
 {% for network in ospf_networks %}
 network {{ network.prefix }} {{ network.wildcard }} area {{ network.area }}
 {% endfor %}
```

Create corresponding tasks in `tasks/main.yml`:

```yaml
---
- name: Configure OSPF
  cisco.ios.ios_config:
    src: ospf.j2
```

### 9. Use Meta Information

Edit `meta/main.yml` to provide information about your role:

```yaml
galaxy_info:
  author: Your Name
  description: Role to configure network devices
  company: Your Company
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: IOS
      versions:
        - all

dependencies: []
```

### 10. Implement Role Dependencies

If your role depends on another role, specify it in `meta/main.yml`:

```yaml
dependencies:
  - role: common_network_config
```

## Advanced Topics
- Using role variables with `vars` and `vars_prompt`
- Implementing conditional role inclusion
- Creating custom modules for use in roles
- Using Ansible Galaxy to share and distribute roles

## Troubleshooting
- Use `ansible-playbook --syntax-check` to verify role syntax
- Enable verbose mode with `-v` when running playbooks to see detailed output
- Check for variable precedence issues if role variables are not applying as expected

## Best Practices
1. Keep roles focused on a single responsibility
2. Use version control for your roles
3. Document your roles thoroughly, including variables and dependencies
4. Use `defaults/` for default variable values and `vars/` for fixed values
5. Leverage community roles from Ansible Galaxy when possible
6. Test roles thoroughly before using in production
7. Use tags in your roles for granular execution control

## Additional Resources
- [Ansible Roles Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Best Practices for Designing Ansible Roles](https://www.ansible.com/blog/ansible-best-practices-essentials)

## Glossary
- **Role**: A way to organize playbooks and other files to facilitate sharing and reuse
- **Task**: A single action to be performed, defined within a role
- **Handler**: A task that only runs when notified by another task
- **Template**: A file containing variables that can be dynamically replaced
- **Meta**: Information about the role, including dependencies
- **Galaxy**: Ansible's hub for sharing roles

