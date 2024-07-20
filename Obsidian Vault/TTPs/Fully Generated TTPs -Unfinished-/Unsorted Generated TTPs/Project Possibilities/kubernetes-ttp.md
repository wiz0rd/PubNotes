# Container Orchestration with Kubernetes TTP

## Project Goal
The goal of this project is to set up and manage a Kubernetes cluster for container orchestration. By the end of this project, you will be able to deploy, scale, and manage containerized applications in a production-like environment, enhancing your skills in modern application deployment and management.

## Overview
This TTP provides detailed instructions for setting up a multi-node Kubernetes cluster and deploying containerized applications.

## Prerequisites
- 3 or more Linux machines (physical or virtual) with at least 2 CPU cores and 2GB RAM each
- Ubuntu 20.04 LTS or later installed on all machines
- Root or sudo access on all machines
- Basic understanding of containerization concepts

## Step-by-step Guide

### 1. Prepare the Nodes

On all nodes:

```bash
# Update the system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Kubernetes repository
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list
sudo apt update

# Install Kubernetes components
sudo apt install -y kubelet kubeadm kubectl

# Hold packages at current version
sudo apt-mark hold kubelet kubeadm kubectl

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Enable kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Add kernel settings
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload sysctl
sudo sysctl --system
```

### 2. Install Container Runtime (containerd)

On all nodes:

```bash
# Install containerd
sudo apt install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
```

### 3. Initialize the Control Plane

On the master node:

```bash
# Initialize the cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Set up kubeconfig for the root user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install Flannel network plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Save the `kubeadm join` command that is output, as you'll need it for joining worker nodes.

### 4. Join Worker Nodes

On each worker node, run the `kubeadm join` command you saved from the previous step. It will look something like this:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 5. Verify Cluster Status

On the master node:

```bash
kubectl get nodes
```

All nodes should show as "Ready".

### 6. Deploy a Sample Application

On the master node:

```bash
# Create a deployment
kubectl create deployment nginx --image=nginx

# Expose the deployment
kubectl expose deployment nginx --port=80 --type=NodePort

# Check the service
kubectl get svc nginx
```

You can now access the nginx web server on any node's IP address at the NodePort assigned.

## Advanced Topics

- Implementing auto-scaling with Horizontal Pod Autoscaler
- Setting up persistent storage with PersistentVolumes
- Configuring Ingress controllers for load balancing
- Using Helm for package management

## Troubleshooting

- If nodes fail to join, check firewall settings and ensure required ports are open
- If pods are stuck in "Pending" state, check for resource constraints or node taints
- For networking issues, verify the CNI plugin (Flannel) is correctly installed

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
