# Day 2: Container Runtime Foundation

## Overview (1.5 hours)
Install and configure container runtimes to understand the foundation layer that Kubernetes uses to run containers.

## Why This Matters for the CKA Exam
- **Container Runtime Interface (CRI)** questions appear in cluster architecture (25% of exam)
- **Troubleshooting runtime issues** is critical for 30% of troubleshooting domain
- **Understanding container lifecycle** helps with workload management (15% of exam)
- **Runtime commands** are essential for debugging pod failures

## Task 1: Install Podman and CRI-O (45 minutes)

### What You'll Do
Install both Podman (for container management practice) and CRI-O (Kubernetes-native runtime).

### Step-by-Step Instructions

#### 1. Install Podman
```bash
# Rocky Linux 8.5 installation
sudo dnf install -y podman

# Verify installation
podman --version
podman info
```

#### 2. Install CRI-O (Kubernetes Container Runtime)
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

#### 3. Configure CRI-O for Kubernetes
```bash
# Configure CRI-O settings
sudo vim /etc/crio/crio.conf

# Key settings to verify/modify:
# [crio.runtime]
# default_runtime = "runc"
# [crio.image]
# default_transport = "docker://"
# [crio.network]
# network_dir = "/etc/cni/net.d/"

# Restart CRI-O after configuration
sudo systemctl restart crio
```

### CKA Exam Connection
- **Runtime Troubleshooting**: Understanding container runtime issues
- **Pod Lifecycle**: How containers start, run, and terminate
- **Cluster Setup**: CRI-O is used in many CKA exam environments
- **Performance**: Runtime selection affects cluster performance

### Validation Commands
```bash
# Test Podman installation
podman run hello-world
podman images
podman ps -a

# Test CRI-O installation
sudo crictl version
sudo crictl info
```

## Task 2: Practice Basic Podman Commands (30 minutes)

### What You'll Do
Master essential container commands that parallel kubectl pod operations.

### Step-by-Step Instructions

#### 1. Container Lifecycle Commands
```bash
# Run a simple container
podman run hello-world

# Run interactive container
podman run -it ubuntu:20.04 /bin/bash
# Inside container, explore:
cat /etc/os-release
ps aux
exit

# Run container in background (detached)
podman run -d --name web-server nginx:latest

# List running containers
podman ps

# List all containers (including stopped)
podman ps -a
```

#### 2. Container Management
```bash
# View container logs
podman logs web-server

# Execute commands in running container
podman exec -it web-server /bin/bash
# Inside container:
ps aux
nginx -t
exit

# Stop container
podman stop web-server

# Start stopped container
podman start web-server

# Remove container
podman rm web-server

# Force remove running container
podman rm -f web-server
```

#### 3. Image Management
```bash
# List images
podman images

# Pull specific image
podman pull nginx:1.21

# Pull image with specific tag
podman pull redis:6.2-alpine

# Remove image
podman rmi hello-world

# Search for images
podman search nginx
```

#### 4. Container Networking and Ports
```bash
# Run container with port mapping
podman run -d -p 8080:80 --name nginx-test nginx:latest

# Test connectivity
curl http://localhost:8080

# Check port bindings
podman port nginx-test

# Clean up
podman rm -f nginx-test
```

### CKA Exam Connection
- **Pod Troubleshooting**: Similar commands exist in kubectl (logs, exec)
- **Container Debugging**: Understanding container internals
- **Image Issues**: Debugging image pull problems
- **Networking**: Port binding concepts apply to Services

### Practice Scenarios
```bash
# Scenario 1: Debug a failing container
podman run -d --name failing-app nginx:nonexistent-tag
podman ps -a  # Check status
podman logs failing-app  # Check logs

# Scenario 2: Inspect running container
podman run -d --name inspect-me nginx:latest
podman inspect inspect-me | grep -A 5 "IPAddress"
podman top inspect-me  # Check processes

# Scenario 3: Container resource usage
podman stats inspect-me  # Monitor resources
```

## Task 3: Understand Container Runtime in Kubernetes Context (15 minutes)

### What You'll Do
Learn how container runtimes integrate with Kubernetes and impact the CKA exam.

### Key Concepts for CKA

#### 1. Container Runtime Interface (CRI)
```bash
# CRI-O is the CRI-compliant runtime
# Kubernetes communicates through CRI

# Key CRI tools for CKA troubleshooting:
sudo crictl --help

# List containers (managed by Kubernetes)
sudo crictl ps

# List images pulled by Kubernetes
sudo crictl images

# Get container logs (for troubleshooting)
sudo crictl logs <container-id>
```

#### 2. Runtime Configuration Impact
```bash
# Runtime configuration affects:
# - Pod startup time
# - Image pulling behavior  
# - Container security
# - Resource management

# Check runtime configuration
sudo cat /etc/crio/crio.conf | grep -A 5 "\[crio.runtime\]"
```

#### 3. Debugging Workflow
```bash
# CKA Troubleshooting Workflow:
# 1. kubectl describe pod <pod-name>
# 2. kubectl logs <pod-name>
# 3. If needed: sudo crictl ps | grep <pod-name>
# 4. If needed: sudo crictl logs <container-id>
```

### CKA Exam Scenarios
1. **Pod Stuck in ContainerCreating**: Runtime configuration issues
2. **Image Pull Errors**: Registry or runtime configuration
3. **Container Crashes**: Runtime or application issues
4. **Performance Problems**: Runtime resource management

### Common CKA Runtime Issues
```bash
# Issue 1: CRI-O not running
sudo systemctl status crio
sudo systemctl start crio

# Issue 2: Runtime configuration errors
sudo journalctl -u crio -f

# Issue 3: Image pull issues
sudo crictl pull nginx:latest
```

## Container vs Pod Concepts

### Understanding the Relationship
| Container (Runtime) | Pod (Kubernetes) |
|-------------------|------------------|
| Single process | One or more containers |
| Podman/Docker manages | kubelet manages |
| Direct runtime commands | kubectl commands |
| Host networking | Pod networking |

### Key Differences for CKA
```bash
# Container commands (direct runtime)
podman run nginx
podman logs <container-id>
podman exec -it <container-id> bash

# Pod commands (through Kubernetes)
kubectl run nginx --image=nginx
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- bash
```

## Troubleshooting Practice

### Common Issues and Solutions

#### Issue 1: CRI-O Service Not Starting
```bash
# Check service status
sudo systemctl status crio

# Check configuration
sudo crio config > /tmp/crio-config.toml
sudo crio --config /tmp/crio-config.toml --debug

# Common fix: Restart after configuration change
sudo systemctl daemon-reload
sudo systemctl restart crio
```

#### Issue 2: Permission Denied with Podman
```bash
# Check user permissions
id
groups

# Add user to required group (if needed)
sudo usermod -aG podman $USER
newgrp podman
```

#### Issue 3: Container Won't Start
```bash
# Check detailed error
podman run --rm -it ubuntu:20.04 /bin/bash

# Check system resources
df -h  # Disk space
free -h  # Memory
```

## Time Management for CKA

### Command Efficiency
- Use `crictl` for Kubernetes-managed containers
- Use `podman` for standalone testing
- Master both `logs` and `exec` commands
- Practice quick image pulling and running

### Exam Strategy
- Always check container runtime if pods fail to start
- Use container debugging as last resort (after kubectl)
- Know the difference between runtime and Kubernetes issues

## Daily Goals Checklist

- [ ] CRI-O installed and running on all nodes
- [ ] Podman commands working correctly
- [ ] Can run, manage, and debug containers
- [ ] Understand runtime vs Kubernetes distinction
- [ ] Practice scenarios completed successfully

## Next Day Preparation

### What You'll Need for Day 3
- Working container runtime ✓
- Understanding of container basics ✓
- Debugging skills with containers ✓

### Coming Up: Kubernetes Architecture
Day 3 will explore how these container runtimes fit into the larger Kubernetes architecture, preparing you for cluster setup.

## Study Notes Section

**Runtime Commands Cheat Sheet:**
```bash
# Quick reference for exam day
podman ps                    # List containers
podman logs <name>          # View logs  
podman exec -it <name> bash # Access container
sudo crictl ps              # List K8s containers
sudo crictl logs <id>       # View K8s container logs
```

**Personal Notes:**
_Record any specific configurations, errors encountered, or additional insights_