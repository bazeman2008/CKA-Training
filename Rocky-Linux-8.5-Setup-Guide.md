# Rocky Linux 8.5 Setup Guide for CKA Training

This guide provides all Rocky Linux 8.5 compatible commands and configurations for the CKA training materials. While the CKA exam uses Ubuntu, this guide helps those practicing on Rocky Linux 8.5.

## Table of Contents
1. [System Setup](#system-setup)
2. [Container Runtime Installation](#container-runtime-installation)
3. [Kubernetes Installation](#kubernetes-installation)
4. [Firewall Configuration](#firewall-configuration)
5. [SELinux Configuration](#selinux-configuration)
6. [Cluster Upgrades](#cluster-upgrades)
7. [Troubleshooting](#troubleshooting)

## System Setup

### Update System and Install Essential Tools
```bash
# Update system packages
sudo dnf update -y

# Install essential tools
sudo dnf install -y vim tmux git curl wget htop net-tools

# Install container runtime dependencies
sudo dnf install -y ca-certificates
# Note: redhat-lsb-core provides lsb-release functionality if needed
sudo dnf install -y redhat-lsb-core

# Install completion package
sudo dnf install -y bash-completion
```

### SSH Configuration
```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Copy to other nodes
ssh-copy-id user@192.168.1.10  # master
ssh-copy-id user@192.168.1.11  # worker-1
ssh-copy-id user@192.168.1.12  # worker-2
```

## Container Runtime Installation

### Why Both Podman and CRI-O?

**Important Distinction:** This guide installs both Podman and CRI-O, which might seem redundant since both are container runtimes. Here's why both are included:

1. **CRI-O** (Required for Kubernetes)
   - The actual container runtime that Kubernetes uses
   - Implements the Container Runtime Interface (CRI)
   - Manages all Kubernetes pod containers
   - Essential for cluster operation
   - Cannot be used standalone easily

2. **Podman** (Training and Troubleshooting Tool)
   - Standalone container management tool
   - Helps learn container concepts before Kubernetes
   - Useful for testing and debugging images
   - Provides direct container access outside of Kubernetes
   - Docker-compatible commands for familiarity

**Production Note:** In a production Kubernetes cluster, you would typically only install CRI-O. Podman is included here specifically for CKA training purposes.

### Install Podman
```bash
# Rocky Linux 8.5 installation
sudo dnf install -y podman

# Verify installation
podman --version
podman info
```

### Install CRI-O
```bash
# Rocky Linux 8.5 installation
export VERSION=1.28      # Match with planned Kubernetes version

# Enable CRI-O module and install
sudo dnf module enable -y cri-o:$VERSION
sudo dnf install -y cri-o

# Note: SELinux considerations for Rocky Linux
# CRI-O works with SELinux enabled, but you may need to:
# - Check SELinux status: getenforce
# - For troubleshooting: sudo setenforce 0 (temporary)
# - For production: Configure proper SELinux contexts

# Start and enable CRI-O
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl status crio
```

## Kubernetes Installation

### Add Kubernetes Repository
```bash
# Rocky Linux 8.5 installation
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

### Install Kubernetes Components
```bash
# Rocky Linux 8.5 installation
sudo dnf install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2 --disableexcludes=kubernetes

# Lock versions to prevent accidental upgrades
sudo dnf install -y yum-plugin-versionlock
sudo dnf versionlock kubelet kubeadm kubectl

# Enable kubelet service (don't start yet - no cluster)
sudo systemctl enable kubelet
```

### Configure Prerequisites
```bash
# Disable swap (required for kubelet)
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

# SELinux configuration for Rocky Linux 8.5
# Check SELinux status
getenforce
# For CRI-O with SELinux, ensure proper contexts are set
# Container runtimes handle SELinux contexts automatically
```

## Firewall Configuration

### Basic Firewall Commands
```bash
# Check firewall status
sudo firewall-cmd --list-all

# Kubernetes required ports - Master Node
sudo firewall-cmd --permanent --add-port=6443/tcp      # API server
sudo firewall-cmd --permanent --add-port=2379-2380/tcp # etcd
sudo firewall-cmd --permanent --add-port=10250/tcp     # Kubelet API
sudo firewall-cmd --permanent --add-port=10251/tcp     # kube-scheduler
sudo firewall-cmd --permanent --add-port=10252/tcp     # kube-controller-manager

# Kubernetes required ports - Worker Nodes
sudo firewall-cmd --permanent --add-port=10250/tcp     # Kubelet API
sudo firewall-cmd --permanent --add-port=30000-32767/tcp # NodePort Services

# Reload firewall
sudo firewall-cmd --reload

# Verify ports
sudo firewall-cmd --list-ports
```

### NodePort Service Configuration
```bash
# Check if NodePort is accessible
sudo firewall-cmd --list-ports | grep <nodeport>

# To open specific NodePort if needed:
sudo firewall-cmd --permanent --add-port=<nodeport>/tcp
sudo firewall-cmd --reload
```

## SELinux Configuration

### SELinux Basics
```bash
# Check SELinux status
getenforce

# View current SELinux mode
sestatus

# Temporary disable for testing (NOT for production)
sudo setenforce 0

# Re-enable enforcing mode
sudo setenforce 1
```

### Common SELinux Configurations for Kubernetes

#### HostPath Volumes
```bash
# Set proper SELinux context for hostPath volumes
sudo semanage fcontext -a -t svirt_sandbox_file_t '/data/hostpath-pv(/.*)?'
sudo restorecon -Rv /data/hostpath-pv
ls -laZ /data/hostpath-pv  # Verify context
```

#### NodePort Services
```bash
# Check SELinux denials for NodePort
sudo ausearch -m AVC | grep node_port

# Allow NodePort range
sudo semanage port -a -t node_port_t -p tcp 30000-32767

# For custom ports
sudo semanage port -a -t http_port_t -p tcp <port>

# Verify
sudo semanage port -l | grep <port>
```

#### Container Runtime
```bash
# For NFS volumes
sudo setsebool -P virt_use_nfs on

# For iSCSI volumes
sudo setsebool -P virt_use_fusefs on

# Check all virtualization booleans
sudo getsebool -a | grep virt
```

### SELinux Troubleshooting
```bash
# View recent SELinux denials
sudo ausearch -m AVC -ts recent

# Check denials for specific service
sudo ausearch -m AVC -ts recent | grep <service-name>

# Generate policy from denials
sudo ausearch -m AVC -ts recent | audit2allow -M mypolicy

# Install generated policy
sudo semodule -i mypolicy.pp

# For development/testing with shared volumes
# Add :z or :Z to volume mounts in pod specs
# :z - shared between containers
# :Z - private unshared label
```

## Cluster Upgrades

### Upgrade Control Plane
```bash
# 1. Update kubeadm
sudo dnf update && sudo dnf install -y kubeadm-1.XX.X

# 2. Check upgrade plan
sudo kubeadm upgrade plan

# 3. Apply upgrade
sudo kubeadm upgrade apply v1.XX.X

# 4. Upgrade kubelet and kubectl
sudo dnf update && sudo dnf install -y kubelet-1.XX.X kubectl-1.XX.X
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Upgrade Worker Nodes
```bash
# 1. From control plane, drain node
kubectl drain <node-name> --ignore-daemonsets

# 2. On worker node, upgrade kubeadm
sudo dnf update && sudo dnf install -y kubeadm-1.XX.X

# 3. Upgrade node configuration
sudo kubeadm upgrade node

# 4. Upgrade kubelet and kubectl
sudo dnf update && sudo dnf install -y kubelet-1.XX.X kubectl-1.XX.X
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# 5. From control plane, uncordon node
kubectl uncordon <node-name>
```

## Troubleshooting

### Common Package Manager Differences
| Ubuntu Command | Rocky Linux 8.5 Command |
|----------------|------------------------|
| apt update | dnf update |
| apt install | dnf install |
| apt-get | dnf |
| apt-mark hold | dnf versionlock |
| apt-cache search | dnf search |

### System Service Management
```bash
# Check service status
sudo systemctl status <service-name>

# View service logs
sudo journalctl -u <service-name> -f

# Restart service
sudo systemctl restart <service-name>
```

### Network Troubleshooting
```bash
# Check listening ports
sudo ss -tlnp

# Check firewall rules
sudo firewall-cmd --list-all

# Check iptables rules
sudo iptables -L -n -v
```

### SELinux Quick Fixes
```bash
# Pod mount permission denied
sudo chcon -Rt svirt_sandbox_file_t /path/to/mount

# Service port access denied
sudo semanage port -a -t http_port_t -p tcp <port>

# Container access to host resources
# Check current context
ls -laZ /path/to/resource

# Set appropriate context
sudo semanage fcontext -a -t container_file_t "/path/to/resource(/.*)?"
sudo restorecon -Rv /path/to/resource
```

## Important Notes

1. **CKA Exam Environment**: The actual CKA exam uses Ubuntu, so familiarize yourself with both Ubuntu and Rocky Linux commands.

2. **SELinux**: Rocky Linux has SELinux enabled by default, which can cause unexpected issues. Always check SELinux when troubleshooting permission problems.

3. **Firewall**: Rocky Linux uses firewalld instead of ufw. Ensure you're comfortable with firewall-cmd commands.

4. **Package Names**: Some packages have different names or version formats between Ubuntu and Rocky Linux.

5. **Repository Configuration**: Repository setup differs significantly between distributions.

## Quick Reference Card

```bash
# Rocky Linux 8.5 Essentials for CKA
# Package Management
sudo dnf update -y
sudo dnf install -y <package>
sudo dnf versionlock <package>

# Firewall
sudo firewall-cmd --permanent --add-port=<port>/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-all

# SELinux
getenforce
sudo semanage fcontext -a -t <context> "/path(/.*)?"
sudo restorecon -Rv /path
sudo ausearch -m AVC -ts recent

# Services
sudo systemctl status <service>
sudo journalctl -u <service> -f
```

This guide should be used alongside the main CKA training materials, substituting commands where necessary for Rocky Linux 8.5 compatibility.