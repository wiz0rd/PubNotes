# Containerlab/Netlab Testing TTP

## Goal
The goal of this TTP is to set up a virtual network testing environment using Containerlab or Netlab, integrate it with Ansible, and use it to validate playbooks before deploying to production networks.

## Overview
This TTP covers the process of setting up a virtual network lab, creating test topologies, and using them to test and validate Ansible playbooks for network automation tasks.

![Containerlab/Netlab Testing Workflow](https://example.com/path/to/containerlab_netlab_workflow.png)
*Figure 1: Containerlab/Netlab Testing Workflow. Replace with an actual diagram showing the testing process flow.*

## Prerequisites
- Linux machine or VM with Docker installed
- Ansible installed and configured
- Basic understanding of network topologies and configurations
- Familiarity with YAML syntax

## Step-by-step Guide

### 1. Install Containerlab

```bash
bash -c "$(curl -sL https://get.containerlab.dev)"
```

Verify the installation:

```bash
containerlab version
```

### 2. Create a Simple Network Topology

Create a file named `simple_topology.yml`:

```yaml
name: simple_topology

topology:
  nodes:
    router1:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    router2:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    switch1:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1

  links:
    - endpoints: ["router1:eth1", "switch1:eth1"]
    - endpoints: ["router2:eth1", "switch1:eth2"]
```

### 3. Deploy the Topology

Deploy the topology using Containerlab:

```bash
sudo containerlab deploy -t simple_topology.yml
```

### 4. Create an Ansible Inventory for the Lab

Create a file named `lab_inventory.yml`:

```yaml
all:
  children:
    lab_routers:
      hosts:
        clab-simple_topology-router1:
          ansible_host: 172.20.20.2
        clab-simple_topology-router2:
          ansible_host: 172.20.20.3
    lab_switches:
      hosts:
        clab-simple_topology-switch1:
          ansible_host: 172.20.20.4
  vars:
    ansible_network_os: nokia.sros.md
    ansible_connection: ansible.netcommon.network_cli
    ansible_user: admin
    ansible_password: admin
```

### 5. Create a Test Playbook

Create a file named `test_playbook.yml`:

```yaml
---
- name: Test Network Configuration
  hosts: all
  gather_facts: no

  tasks:
    - name: Configure hostname
      nokia.sros.config:
        lines:
          - configure system name {{ inventory_hostname }}

    - name: Configure interfaces
      nokia.sros.config:
        lines:
          - configure port 1/1/1 admin-state enable
          - configure port 1/1/1 description "Connected to {{ item }}"
      loop: "{{ play_hosts | difference([inventory_hostname]) }}"

    - name: Verify configuration
      nokia.sros.command:
        commands:
          - show system information
          - show port 1/1/1
      register: config_check

    - name: Display configuration
      debug:
        var: config_check.stdout_lines
```

### 6. Run the Test Playbook

Execute the test playbook against the lab environment:

```bash
ansible-playbook -i lab_inventory.yml test_playbook.yml
```

### 7. Implement Automated Testing

Create a file named `run_tests.sh`:

```bash
#!/bin/bash

# Deploy the lab topology
sudo containerlab deploy -t simple_topology.yml

# Wait for nodes to be ready
sleep 60

# Run the test playbook
ansible-playbook -i lab_inventory.yml test_playbook.yml

# Destroy the lab topology
sudo containerlab destroy -t simple_topology.yml
```

Make the script executable:

```bash
chmod +x run_tests.sh
```

### 8. Integrate with CI/CD Pipeline

Add the following stage to your CI/CD pipeline (e.g., in GitLab CI):

```yaml
test_network_playbooks:
  stage: test
  script:
    - ./run_tests.sh
  only:
    - merge_requests
```

### 9. Create More Complex Topologies

Create a file named `complex_topology.yml`:

```yaml
name: complex_topology

topology:
  nodes:
    spine1:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    spine2:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    leaf1:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    leaf2:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    leaf3:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1
    leaf4:
      kind: nokia_sr
      image: vrnetlab/vr-sros:21.2.R1

  links:
    - endpoints: ["spine1:eth1", "leaf1:eth1"]
    - endpoints: ["spine1:eth2", "leaf2:eth1"]
    - endpoints: ["spine1:eth3", "leaf3:eth1"]
    - endpoints: ["spine1:eth4", "leaf4:eth1"]
    - endpoints: ["spine2:eth1", "leaf1:eth2"]
    - endpoints: ["spine2:eth2", "leaf2:eth2"]
    - endpoints: ["spine2:eth3", "leaf3:eth2"]
    - endpoints: ["spine2:eth4", "leaf4:eth2"]
```

### 10. Implement Network Validation Tests

Create a file named `validate_network.yml`:

```yaml
---
- name: Validate Network Configuration
  hosts: all
  gather_facts: no

  tasks:
    - name: Check OSPF neighbors
      nokia.sros.command:
        commands:
          - show router ospf neighbor
      register: ospf_check

    - name: Verify OSPF neighbors
      assert:
        that:
          - "'Full' in ospf_check.stdout[0]"
        fail_msg: "OSPF neighbor check failed on {{ inventory_hostname }}"

    - name: Check BGP peers
      nokia.sros.command:
        commands:
          - show router bgp summary
      register: bgp_check

    - name: Verify BGP peers
      assert:
        that:
          - "'Established' in bgp_check.stdout[0]"
        fail_msg: "BGP peer check failed on {{ inventory_hostname }}"

    - name: Ping test
      nokia.sros.command:
        commands:
          - ping {{ item }} count 5
      loop: "{{ groups['all'] | difference([inventory_hostname]) }}"
      register: ping_results

    - name: Verify ping results
      assert:
        that:
          - "'0% packet loss' in item.stdout[0]"
        fail_msg: "Ping test failed from {{ inventory_hostname }} to {{ item.item }}"
      loop: "{{ ping_results.results }}"
```

## Advanced Topics
- Implementing network chaos testing in the lab environment
- Creating custom network device images for Containerlab
- Automating the generation of test traffic in the lab
- Integrating performance testing tools with the virtual lab

## Troubleshooting
- Verify that Docker is running and Containerlab is correctly installed
- Check Containerlab logs for deployment issues
- Ensure that the virtual network devices are reachable from the Ansible control node
- Review Ansible logs for detailed error messages during playbook execution
- Use `--check` mode to test playbooks without making changes

## Best Practices
1. Version control your lab topologies and test playbooks
2. Use CI/CD pipelines to automatically test playbooks on every change
3. Create lab topologies that closely mimic your production environment
4. Implement comprehensive validation tests for your network configurations
5. Regularly update your lab environment to match production software versions
6. Document your test scenarios and expected results
7. Use consistent naming conventions for lab devices and interfaces

## Additional Resources
- [Containerlab Documentation](https://containerlab.srlinux.dev/)
- [Netlab Documentation](https://netlab.tools/)
- [Ansible Network Automation Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [Network Test Automation (Book)](https://www.oreilly.com/library/view/network-test-automation/9781492071174/)

## Glossary
- **Containerlab**: A tool for orchestrating and managing container-based network labs
- **Netlab**: A Python library for automating network labs, supporting various virtualization platforms
- **Topology**: The arrangement of nodes and their interconnections in a network
- **CI/CD**: Continuous Integration/Continuous Deployment, a method to frequently deliver apps to customers
- **OSPF**: Open Shortest Path First, a routing protocol for IP networks
- **BGP**: Border Gateway Protocol, a standardized exterior gateway protocol designed to exchange routing and reachability information
- **Ping**: A computer network administration software utility used to test the reachability of a host on an Internet Protocol network
