# Day 5: First Cluster Setup

## Overview (1.5 hours)
Create your first Kubernetes cluster using kubeadm and understand the initialization process that's critical for CKA exam success.

## Why This Matters for the CKA Exam
- **Cluster setup is guaranteed** to appear on the CKA exam
- **kubeadm commands** are essential for cluster management
- **Certificate understanding** helps with troubleshooting
- **Initialization process** knowledge enables cluster recovery

## Task 1: Complete kubeadm init Process (60 minutes)

### What You'll Do
Initialize a Kubernetes cluster on the master node and understand each step of the process.

### Pre-Initialization Checks

#### 1. Verify Prerequisites
```bash
# Check all nodes are ready
ping worker-node-1
ping worker-node-2
ssh worker-node-1 "echo 'Node 1 accessible'"
ssh worker-node-2 "echo 'Node 2 accessible'"

# Verify Kubernetes components installed
kubeadm version
kubelet --version
kubectl version --client

# Check container runtime
sudo systemctl status crio
sudo crictl version

# Verify system settings
sudo sysctl net.bridge.bridge-nf-call-iptables  # Should be 1
free -h | grep Swap  # Should show 0 for swap
```

#### 2. Configure Container Runtime
```bash
# Ensure CRI-O is configured properly
sudo systemctl enable crio --now
sudo systemctl status crio

# Configure kubelet to use CRI-O
sudo mkdir -p /etc/systemd/system/kubelet.service.d
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service.d/0-crio.conf
[Service]
Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///var/run/crio/crio.sock"
EOF

sudo systemctl daemon-reload
```

### Cluster Initialization

#### 1. Initialize the Master Node
```bash
# Initialize cluster with specific pod network CIDR
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --control-plane-endpoint=$(hostname -i) \
  --upload-certs

# Expected output includes:
# - Pre-flight checks
# - Certificate generation
# - Control plane component startup
# - Join commands for worker nodes
```

#### 2. Configure kubectl for Regular User
```bash
# Set up kubectl configuration (SAVE THE JOIN COMMAND!)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Test cluster access
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

#### 3. Install CNI Network Plugin
```bash
# Install Flannel CNI (matches pod-network-cidr)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Alternative: Install Calico CNI
# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml

# Wait for CNI pods to be ready
kubectl get pods -n kube-flannel
kubectl get pods -n kube-system | grep coredns
```

#### 4. Verify Control Plane
```bash
# Check all system pods are running
kubectl get pods -n kube-system

# Verify control plane components
kubectl get componentstatuses
kubectl cluster-info

# Check node status (should be Ready)
kubectl get nodes
```

### Understanding kubeadm init Output

#### 1. Pre-flight Checks
```bash
# kubeadm validates:
# - System requirements (CPU, memory, swap)
# - Port availability (6443, 2379-2380, 10250-10252)
# - Container runtime connectivity
# - Kubernetes component versions
```

#### 2. Certificate Generation
```bash
# Certificates created in /etc/kubernetes/pki/
ls -la /etc/kubernetes/pki/

# Key certificates:
# - ca.crt/ca.key - Cluster CA
# - apiserver.crt/apiserver.key - API server
# - etcd/ca.crt - etcd CA
# - front-proxy-ca.crt - Front proxy CA

# Check certificate validity
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A 2 "Validity"
```

#### 3. Static Pod Manifests
```bash
# Control plane components as static pods
ls -la /etc/kubernetes/manifests/

# Files created:
# - kube-apiserver.yaml
# - kube-controller-manager.yaml  
# - kube-scheduler.yaml
# - etcd.yaml

# View a manifest
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | head -20
```

### CKA Exam Application

#### Critical kubeadm Commands for Exam
```bash
# Initialize cluster (basic)
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Initialize with specific options
sudo kubeadm init \
  --kubernetes-version=v1.28.2 \
  --pod-network-cidr=10.244.0.0/16 \
  --service-cidr=10.96.0.0/12 \
  --control-plane-endpoint=master.local

# Get join command for worker nodes
kubeadm token create --print-join-command

# Generate new bootstrap tokens
kubeadm token generate
kubeadm token create <token> --print-join-command --ttl=24h
```

## Task 2: Understand Cluster Certificates and Tokens (20 minutes)

### What You'll Do
Learn about the certificate and token system that enables secure cluster communication.

### Certificate Authority (CA) Structure

#### 1. Cluster CA Certificates
```bash
# Main cluster CA
sudo openssl x509 -in /etc/kubernetes/pki/ca.crt -text -noout | grep -A 2 "Subject:"

# etcd CA (separate for etcd communication)
sudo openssl x509 -in /etc/kubernetes/pki/etcd/ca.crt -text -noout | grep -A 2 "Subject:"

# Front Proxy CA (for extension API servers)
sudo openssl x509 -in /etc/kubernetes/pki/front-proxy-ca.crt -text -noout | grep -A 2 "Subject:"
```

#### 2. Component Certificates
```bash
# API Server certificate
sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A 5 "DNS:"

# Kubelet client certificate
sudo openssl x509 -in /etc/kubernetes/pki/apiserver-kubelet-client.crt -text -noout | grep "Subject:"

# Check certificate expiration (important for troubleshooting)
sudo kubeadm certs check-expiration
```

### Bootstrap Tokens

#### 1. Understanding Bootstrap Tokens
```bash
# List current tokens
kubeadm token list

# Token format: [a-z0-9]{6}.[a-z0-9]{16}
# First part: Token ID
# Second part: Token Secret

# Tokens are stored as secrets in kube-system namespace
kubectl get secrets -n kube-system | grep bootstrap-token
```

#### 2. Token Management
```bash
# Create new token
kubeadm token generate
TOKEN=$(kubeadm token generate)
kubeadm token create $TOKEN --description "Token for worker node join" --ttl 24h

# Delete expired tokens
kubeadm token delete <token-id>

# Get CA certificate hash (needed for secure join)
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* /sha256:/'
```

### CKA Exam Scenarios

#### 1. Certificate Expiration Issues
```bash
# Problem: API server not starting due to expired certificates
# Solution:
kubeadm certs check-expiration
kubeadm certs renew all
sudo systemctl restart kubelet
```

#### 2. Worker Node Join Issues
```bash
# Problem: Worker node can't join cluster
# Solution: Generate new join command
kubeadm token create --print-join-command

# Or manually construct:
kubeadm join <master-ip>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

## Task 3: Document Setup Steps for Speed Improvement (10 minutes)

### What You'll Do
Create a cluster setup checklist that you can execute quickly during the exam.

### Quick Cluster Setup Checklist

#### 1. Pre-Setup (5 minutes)
```bash
# [ ] Verify 3 nodes accessible
# [ ] Check kubeadm/kubelet/kubectl installed
# [ ] Verify container runtime running
# [ ] Confirm swap disabled
# [ ] Check required ports available
```

#### 2. Master Node Setup (10 minutes)
```bash
# [ ] Run kubeadm init with pod-network-cidr
# [ ] Copy admin.conf to ~/.kube/config
# [ ] Install CNI plugin (Flannel/Calico)
# [ ] Verify control plane pods running
# [ ] Save join command for workers
```

#### 3. Worker Node Setup (5 minutes)
```bash
# [ ] Execute join command on each worker
# [ ] Verify nodes show as Ready
# [ ] Test pod scheduling on workers
```

### Speed Optimization Commands

#### 1. One-liner Cluster Init
```bash
# Combined initialization and setup
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 && \
mkdir -p $HOME/.kube && \
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config && \
sudo chown $(id -u):$(id -g) $HOME/.kube/config && \
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

#### 2. Quick Verification Script
```bash
# Create verification script
cat > verify_cluster.sh << 'EOF'
#!/bin/bash
echo "=== Cluster Info ==="
kubectl cluster-info

echo "=== Nodes ==="
kubectl get nodes

echo "=== System Pods ==="
kubectl get pods -n kube-system

echo "=== Component Status ==="
kubectl get componentstatuses
EOF

chmod +x verify_cluster.sh
./verify_cluster.sh
```

### Common Issues and Quick Fixes

#### 1. kubeadm init Failures
```bash
# Problem: Init fails due to port conflicts
# Check: netstat -tulpn | grep :6443
# Fix: Kill process using port or use different endpoint

# Problem: Container runtime not detected
# Check: sudo systemctl status crio
# Fix: sudo systemctl start crio

# Problem: Swap not disabled
# Check: free -h
# Fix: sudo swapoff -a
```

#### 2. CNI Installation Issues
```bash
# Problem: CoreDNS pods stuck in Pending
# Check: kubectl describe pod -n kube-system coredns-<id>
# Fix: Ensure CNI plugin installed correctly

# Problem: Flannel pods not starting
# Check: kubectl logs -n kube-flannel kube-flannel-<id>
# Fix: Verify pod-network-cidr matches Flannel config
```

## CKA Exam Time Management

### Target Times for Cluster Setup
- **Basic cluster init**: 15 minutes
- **Complete setup with CNI**: 20 minutes
- **Verification and testing**: 5 minutes
- **Total cluster setup**: 25 minutes maximum

### Exam Strategy Tips
1. **Always note the join command** - you'll need it for worker nodes
2. **Install CNI immediately** - control plane won't be fully ready without it
3. **Verify each step** - don't proceed until current step is working
4. **Practice until automatic** - muscle memory is crucial under exam pressure

## Daily Goals Checklist

- [ ] Successfully initialized Kubernetes cluster
- [ ] CNI plugin installed and working
- [ ] All control plane pods running
- [ ] kubectl working correctly
- [ ] Understand certificate and token system
- [ ] Created speed optimization checklist
- [ ] Cluster setup time under 25 minutes

## Weekend Session Preparation

### What You'll Practice This Weekend
- Multiple cluster setups from scratch
- Worker node joining
- Basic troubleshooting
- Speed optimization

### Skills to Master
- Cluster setup in under 20 minutes
- Troubleshooting common init failures
- Understanding certificate locations
- Quick cluster verification

## Study Notes Section

**Quick Setup Commands:**
```bash
# Master init
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# kubectl setup
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# CNI install
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Get join command
kubeadm token create --print-join-command
```

**Personal Setup Notes:**
_Document any environment-specific configurations or issues encountered_

**Certificate Paths:**
- CA: /etc/kubernetes/pki/ca.crt
- API Server: /etc/kubernetes/pki/apiserver.crt
- etcd: /etc/kubernetes/pki/etcd/
- Admin config: /etc/kubernetes/admin.conf