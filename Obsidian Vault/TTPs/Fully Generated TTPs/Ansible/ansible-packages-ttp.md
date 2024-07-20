# Ansible Packages TTP

## Goal
The goal of this TTP is to provide a comprehensive understanding of Ansible packages, including how to manage, create, and share them. This knowledge will enable you to efficiently organize and distribute your Ansible code for network automation tasks.

## Overview
Ansible packages, often referred to as collections, are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins. This TTP covers how to use existing packages, create your own, and share them with the community.

![Ansible Packages Overview](https://example.com/path/to/ansible_packages_overview.png)
*Figure 1: Ansible Packages Overview. Replace with an actual diagram showing the structure and components of Ansible packages/collections.*

## Prerequisites
- Basic understanding of Ansible (covered in the Ansible Basics TTP)
- Familiarity with Ansible roles (covered in the Ansible Roles TTP)
- Basic understanding of Python (for creating custom modules)

## Step-by-step Guide

### 1. Understanding Ansible Collections

Ansible Collections are a packaging format for bundling and distributing Ansible content. They can include:

- Modules
- Roles
- Plugins
- Playbooks
- Documentation

### 2. Installing Ansible Collections

Use `ansible-galaxy` to install collections:

```bash
ansible-galaxy collection install cisco.ios
```

To install a specific version:

```bash
ansible-galaxy collection install cisco.ios:2.0.0
```

### 3. Using Installed Collections

In your playbooks, reference the collection:

```yaml
---
- name: Configure Cisco IOS devices
  hosts: cisco_devices
  tasks:
    - name: Configure OSPF
      cisco.ios.ios_ospf_interfaces:
        config:
          - name: GigabitEthernet0/0
            ospf:
              area_id: 0
              network:
                point_to_point: true
```

### 4. Creating a Custom Collection

Create a new collection:

```bash
ansible-galaxy collection init my_namespace.my_collection
```

This creates a directory structure:

```
my_namespace/my_collection/
├── README.md
├── galaxy.yml
├── plugins/
│   └── README.md
├── roles/
└── playbooks/
```

### 5. Adding Content to Your Collection

Add roles, modules, and playbooks to their respective directories. For example, create a custom module in `plugins/modules/my_custom_module.py`:

```python
#!/usr/bin/python

from ansible.module_utils.basic import AnsibleModule

def main():
    module = AnsibleModule(
        argument_spec=dict(
            name=dict(type='str', required=True),
            state=dict(type='str', choices=['present', 'absent'], default='present'),
        )
    )

    name = module.params['name']
    state = module.params['state']

    # Your module logic here

    module.exit_json(changed=True, name=name, state=state)

if __name__ == '__main__':
    main()
```

### 6. Building Your Collection

Update the `galaxy.yml` file with your collection's metadata:

```yaml
namespace: my_namespace
name: my_collection
version: 1.0.0
readme: README.md
authors:
  - Your Name <your.email@example.com>
description: A collection for network automation
license:
  - GPL-2.0-or-later
tags:
  - networking
  - cisco
repository: http://example.com/repository
documentation: http://docs.example.com
homepage: http://example.com
issues: http://example.com/issue/tracker
```

Build the collection:

```bash
ansible-galaxy collection build
```

### 7. Installing Your Collection

Install your built collection:

```bash
ansible-galaxy collection install my_namespace-my_collection-1.0.0.tar.gz
```

### 8. Publishing Your Collection

To share your collection on Ansible Galaxy:

1. Create an account on [Ansible Galaxy](https://galaxy.ansible.com/)
2. Obtain an API key from your Galaxy profile
3. Publish your collection:

```bash
ansible-galaxy collection publish ./my_namespace-my_collection-1.0.0.tar.gz --api-key=your_api_key
```

### 9. Using ansible-galaxy for Roles

While collections are the newer format, roles are still widely used. Manage roles with `ansible-galaxy`:

Install a role:
```bash
ansible-galaxy install username.role_name
```

Create a new role:
```bash
ansible-galaxy init my_new_role
```

### 10. Creating Requirements Files

Create a `requirements.yml` file to specify dependencies:

```yaml
---
collections:
  - name: cisco.ios
    version: '>=2.0.0'
  - name: my_namespace.my_collection
    source: https://galaxy.ansible.com

roles:
  - name: geerlingguy.nginx
    version: 2.8.0
```

Install all requirements:

```bash
ansible-galaxy install -r requirements.yml
```

## Advanced Topics
- Creating custom plugins (e.g., filter plugins, lookup plugins)
- Integrating tests into your collection
- Using collection-level variables and defaults
- Implementing cross-platform support in your collection

## Troubleshooting
- Use `ansible-galaxy collection list` to see installed collections
- Check the Ansible configuration file for the collections path
- Verify that the collection is in the Ansible collection path when importing
- Use verbose mode (`-vvv`) when running playbooks to debug collection-related issues

## Best Practices
1. Version your collections properly following semantic versioning
2. Include comprehensive documentation with your collection
3. Test your collection thoroughly before publishing
4. Use tags to make your collection discoverable
5. Keep your published collections up to date
6. Use `ansible-lint` to ensure your collection follows best practices

## Additional Resources
- [Ansible Collections Documentation](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)
- [Developing Ansible Collections](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections.html)
- [Ansible Galaxy User Guide](https://galaxy.ansible.com/docs/)

## Glossary
- **Collection**: A distribution format for Ansible content that can include playbooks, roles, modules, and plugins
- **Namespace**: A unique identifier for a group of collections, typically an organization name
- **Galaxy**: Ansible's hub for sharing Ansible content
- **Requirements file**: A YAML file specifying Ansible dependencies (roles and collections)
- **Plugin**: A piece of code that extends Ansible's core functionality
- **Module**: A unit of code executed by Ansible on the managed nodes

