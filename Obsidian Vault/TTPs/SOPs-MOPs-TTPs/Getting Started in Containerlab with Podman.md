# Getting Started with Containerlab using Podman on Red Hat

## Introduction

This guide walks you through setting up a three-router lab using Containerlab with Podman on a Red Hat system. We'll create a topology with a Cisco CSR 1000v, a Juniper vJunos switch, and a Cisco Nexus 9000v, each connected to the others with two links.

## Prerequisites

- Red Hat Linux system
- Podman installed
- Containerlab installed
- Router images for CSR 1000v, vJunos switch, and Nexus 9000v

## Step 1: Verify Available Images

First, check the available images in your Podman repository:

```bash
podman images
```

You should see entries for the required router images. Example output:

```
REPOSITORY                          TAG         IMAGE ID      CREATED      SIZE
localhost/vrnetlab/vr-csr           17.03.08    c8026f5a4463  2 weeks ago  1.86 GB
localhost/vrnetlab/vr-n9kv          10.3.5      a23b418f0bdc  2 weeks ago  2.89 GB
localhost/vrnetlab/vr-vjunosswitch  23.2R1.14   d78adb3d5077  2 weeks ago  4.91 GB
```

## Step 2: Create the Topology File

Create a file named `podman-topology.yml` with the following content:

```yaml
name: three-router-topology
mgmt:
  network: three-router-mgmt
  ipv4-subnet: 172.100.100.0/24
topology:
  nodes:
    vjunos-switch:
      kind: juniper_vjunosswitch
      image: localhost/vrnetlab/vr-vjunosswitch:23.2R1.14
      mgmt-ipv4: 172.100.100.10
    csr-router:
      kind: vr-csr
      image: localhost/vrnetlab/vr-csr:17.03.08
      mgmt-ipv4: 172.100.100.20
    n9kv-switch:
      kind: vr-n9kv
      image: localhost/vrnetlab/vr-n9kv:10.3.5
      mgmt-ipv4: 172.100.100.30
  links:
    - endpoints: ["vjunos-switch:eth1", "csr-router:eth1"]
    - endpoints: ["vjunos-switch:eth2", "csr-router:eth2"]
    - endpoints: ["vjunos-switch:eth3", "n9kv-switch:eth1"]
    - endpoints: ["vjunos-switch:eth4", "n9kv-switch:eth2"]
    - endpoints: ["csr-router:eth3", "n9kv-switch:eth3"]
    - endpoints: ["csr-router:eth4", "n9kv-switch:eth4"]
```

This topology creates three nodes (vjunos-switch, csr-router, and n9kv-switch) and connects them with two links each.

## Step 3: Deploy the Lab

Deploy the lab using Containerlab with Podman:

```bash
sudo containerlab deploy --runtime podman -t podman-topology.yml
```

If you need more detailed output, add the `--debug` flag:

```bash
sudo containerlab deploy --runtime podman -t podman-topology.yml --debug
```

## Step 4: Verify Deployment

After deployment, Containerlab will display a summary of the deployed nodes, including their management IP addresses and states.

## Step 5: Accessing the Routers

You can access the routers using their management IP addresses:

- vJunos Switch: 172.100.100.10
- CSR Router: 172.100.100.20
- N9KV Switch: 172.100.100.30

Use SSH to connect, for example:

```bash
ssh admin@172.100.100.20
```

Default credentials are typically:
- Username: admin
- Password: admin

(Note: Actual credentials may vary based on your image configuration)

## Step 6: Destroying the Lab

When you're done, you can destroy the lab using:

```bash
sudo containerlab destroy --runtime podman -t podman-topology.yml
```

## Conclusion

You've now set up a three-router lab using Containerlab with Podman on Red Hat. This setup provides a great foundation for network testing and learning. Remember to consult the Containerlab and Podman documentation for more advanced features and configurations.
