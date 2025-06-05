# Day 4: kubectl Installation & Basics

## Overview (1.5 hours)
Install kubectl and master the fundamental commands that form the foundation of every CKA exam question.

## Why This Matters for the CKA Exam
- **100% of exam questions** require kubectl commands
- **Command efficiency** directly impacts time management
- **Output formats** are crucial for troubleshooting tasks
- **kubectl explain** saves time during the exam by providing API documentation

## Task 1: Install kubeadm, kubelet, kubectl on All Nodes (45 minutes)

### What You'll Do
Install the essential Kubernetes components needed for cluster creation and management.

### Step-by-Step Installation

#### 1. Add Kubernetes Repository
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

#### 2. Install Kubernetes Components
```bash
# Rocky Linux 8.5 installation
sudo dnf install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2 --disableexcludes=kubernetes

# Lock versions to prevent accidental upgrades
sudo dnf install -y yum-plugin-versionlock
sudo dnf versionlock kubelet kubeadm kubectl

# Enable kubelet service (don't start yet - no cluster)
sudo systemctl enable kubelet
```

#### 3. Configure Prerequisites
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

#### 4. Verify Installation
```bash
# Check versions (should all be 1.28.2)
kubeadm version
kubelet --version
kubectl version --client

# Check kubelet service (should be inactive but enabled)
sudo systemctl status kubelet
```

#### 5. Install on All Nodes
```bash
# Copy installation script to other nodes
cat > install_k8s.sh << 'EOF'
#!/bin/bash
# [Insert the above installation commands]
EOF

chmod +x install_k8s.sh
scp install_k8s.sh worker-node-1:~/
scp install_k8s.sh worker-node-2:~/

# Execute on worker nodes
ssh worker-node-1 "sudo ./install_k8s.sh"
ssh worker-node-2 "sudo ./install_k8s.sh"
```

### CKA Exam Connection
- **Cluster Setup Questions**: kubeadm is used to initialize clusters
- **Node Management**: Understanding kubelet service
- **Version Compatibility**: Knowing component versions for troubleshooting

## Task 2: Practice Basic kubectl Commands and Output Formats (30 minutes)

### What You'll Do
Master essential kubectl commands and output formats used in every exam question.

### Basic Command Structure
```bash
# kubectl command syntax
kubectl [command] [type] [name] [flags]

# Examples:
kubectl get pods                    # List pods
kubectl describe pod nginx         # Show pod details
kubectl delete deployment webapp   # Delete deployment
kubectl apply -f manifest.yaml     # Apply configuration
```

### Essential kubectl Commands

#### 1. Resource Management
```bash
# Create resources
kubectl create deployment nginx --image=nginx
kubectl run test-pod --image=busybox --command -- sleep 3600

# Apply configurations
kubectl apply -f pod.yaml
kubectl apply -f . --recursive

# Delete resources
kubectl delete pod test-pod
kubectl delete deployment nginx
kubectl delete -f pod.yaml

# Scale resources
kubectl scale deployment nginx --replicas=3
```

#### 2. Resource Inspection
```bash
# List resources
kubectl get pods
kubectl get nodes
kubectl get deployments
kubectl get services
kubectl get all

# Detailed information
kubectl describe pod nginx
kubectl describe node worker-node-1
kubectl describe deployment webapp

# Resource logs
kubectl logs pod-name
kubectl logs deployment/nginx
kubectl logs -f pod-name  # Follow logs
```

#### 3. Resource Editing
```bash
# Edit resources in place
kubectl edit pod nginx
kubectl edit deployment webapp
kubectl edit service api

# Patch resources
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'
```

### Output Formats (Critical for CKA)

#### 1. YAML Output (-o yaml)
```bash
# Get full YAML configuration
kubectl get pod nginx -o yaml

# Generate YAML template
kubectl create deployment test --image=nginx --dry-run=client -o yaml

# Save configuration for backup
kubectl get deployment webapp -o yaml > webapp-backup.yaml
```

#### 2. JSON Output (-o json)
```bash
# Get JSON format
kubectl get pod nginx -o json

# Extract specific fields with JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
```

#### 3. Custom Columns (-o custom-columns)
```bash
# Custom output format
kubectl get pods -o custom-columns="NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName"

kubectl get nodes -o custom-columns="NAME:.metadata.name,STATUS:.status.conditions[?(@.type=='Ready')].status"
```

#### 4. Wide Output (-o wide)
```bash
# Extended information
kubectl get pods -o wide
kubectl get nodes -o wide
kubectl get services -o wide
```

### Field Selectors and Label Selectors

#### 1. Field Selectors
```bash
# Filter by field values
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector spec.nodeName=worker-node-1
kubectl get events --field-selector involvedObject.name=nginx
```

#### 2. Label Selectors
```bash
# Filter by labels
kubectl get pods -l app=nginx
kubectl get pods -l 'environment in (production,staging)'
kubectl get pods -l app=nginx,version=v1

# Show labels
kubectl get pods --show-labels
```

### CKA Exam Application
```bash
# Common exam patterns:
# 1. Find all pods in specific namespace
kubectl get pods -n kube-system

# 2. Get pod running on specific node
kubectl get pods -o wide --field-selector spec.nodeName=worker-node-1

# 3. Generate YAML for modification
kubectl create deployment test --image=nginx --dry-run=client -o yaml > test.yaml

# 4. Extract specific information
kubectl get pods -o jsonpath='{.items[0].status.podIP}'
```

## Task 3: Master kubectl explain for API Reference (15 minutes)

### What You'll Do
Learn to use kubectl explain as your API documentation during the exam.

### Using kubectl explain

#### 1. Basic Usage
```bash
# Get help for resource types
kubectl explain pod
kubectl explain deployment
kubectl explain service
kubectl explain node

# Get help for specific fields
kubectl explain pod.spec
kubectl explain deployment.spec.template
kubectl explain service.spec.ports
```

#### 2. Nested Field Exploration
```bash
# Explore container specifications
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources
kubectl explain pod.spec.containers.env

# Explore deployment specifications
kubectl explain deployment.spec.strategy
kubectl explain deployment.spec.template.spec
```

#### 3. Advanced Field Discovery
```bash
# Find all available fields
kubectl explain pod --recursive
kubectl explain deployment --recursive | grep -A 5 "strategy"

# Get specific field documentation
kubectl explain pod.spec.securityContext
kubectl explain pod.spec.affinity.nodeAffinity
```

### CKA Exam Usage Patterns

#### 1. Quick Syntax Check
```bash
# When you need YAML structure
kubectl explain pod.spec.volumes
kubectl explain persistentvolume.spec

# When you need field names
kubectl explain networkpolicy.spec.ingress
kubectl explain configmap.data
```

#### 2. Complex Configuration Help
```bash
# For advanced pod configurations
kubectl explain pod.spec.containers.livenessProbe
kubectl explain pod.spec.initContainers

# For security contexts
kubectl explain pod.spec.securityContext
kubectl explain pod.spec.containers.securityContext
```

#### 3. Resource Relationship Understanding
```bash
# Understanding service specifications
kubectl explain service.spec.selector
kubectl explain service.spec.ports

# Understanding storage
kubectl explain persistentvolumeclaim.spec
kubectl explain storageclass.parameters
```

### Time-Saving Exam Tips

#### 1. Combine with --dry-run
```bash
# Generate template then use explain for fields
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl explain deployment.spec.strategy  # To modify strategy
```

#### 2. Quick Field Lookup
```bash
# Instead of remembering exact syntax:
kubectl explain pod.spec.containers.ports  # Get port structure
kubectl explain pod.spec.tolerations      # Get toleration syntax
```

#### 3. Validation Before Apply
```bash
# Check if field names are correct
kubectl explain pod.spec.nodeSelector
kubectl apply -f pod.yaml --dry-run=client  # Validate syntax
```

## Practice Scenarios for CKA Preparation

### Scenario 1: Resource Creation and Inspection
```bash
# Create a deployment
kubectl create deployment web --image=nginx:1.20

# Scale it
kubectl scale deployment web --replicas=3

# Inspect it
kubectl get deployment web -o yaml
kubectl describe deployment web
kubectl get pods -l app=web -o wide

# Clean up
kubectl delete deployment web
```

### Scenario 2: YAML Generation and Modification
```bash
# Generate pod YAML
kubectl run test-pod --image=busybox --command --dry-run=client -o yaml -- sleep 3600 > pod.yaml

# Edit the YAML (add resources)
kubectl explain pod.spec.containers.resources
vim pod.yaml  # Add resource requests/limits

# Apply and verify
kubectl apply -f pod.yaml
kubectl describe pod test-pod | grep -A 5 "Requests"
```

### Scenario 3: Troubleshooting with kubectl
```bash
# Create a problematic pod
kubectl run broken-pod --image=nginx:nonexistent

# Troubleshoot
kubectl get pods broken-pod
kubectl describe pod broken-pod
kubectl get events --field-selector involvedObject.name=broken-pod

# Fix and verify
kubectl delete pod broken-pod
kubectl run fixed-pod --image=nginx
kubectl get pods fixed-pod -o wide
```

## Common kubectl Patterns for CKA

### 1. Quick Resource Operations
```bash
# Create and expose in one go
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort

# Generate, edit, apply pattern
kubectl create configmap app-config --from-literal=key=value --dry-run=client -o yaml > config.yaml
# Edit config.yaml
kubectl apply -f config.yaml
```

### 2. Debugging Workflows
```bash
# Standard debugging sequence
kubectl get pods                           # List pods
kubectl describe pod <problematic-pod>     # Get detailed info
kubectl logs <problematic-pod>            # Check logs
kubectl get events | grep <pod-name>      # Check events
```

### 3. Information Extraction
```bash
# Get specific information quickly
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o custom-columns="NAME:.metadata.name,VERSION:.status.nodeInfo.kubeletVersion"
```

## Daily Goals Checklist

- [ ] Kubernetes components installed on all nodes
- [ ] kubectl commands working correctly
- [ ] Familiar with all major output formats
- [ ] Can use kubectl explain effectively
- [ ] Practice scenarios completed successfully
- [ ] Ready for cluster creation tomorrow

## Next Day Preparation

### What You'll Need for Day 5
- kubectl installed and working ✓
- Basic command proficiency ✓
- Output format mastery ✓
- kubectl explain skills ✓

### Coming Up: First Cluster Setup
Day 5 will use kubeadm to create your first Kubernetes cluster, putting all the preparation together.

## Study Notes Section

**Essential kubectl Commands:**
```bash
# Quick reference for daily use
kubectl get pods -o wide --show-labels
kubectl describe pod <name>
kubectl logs <pod> -f
kubectl explain <resource>.<field>
kubectl create <resource> --dry-run=client -o yaml
kubectl get events --sort-by=.metadata.creationTimestamp
```

**Personal Command Notes:**
_Record any kubectl patterns or shortcuts you discover_