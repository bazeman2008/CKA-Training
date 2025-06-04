# Week 1: Environment & Container Fundamentals

## Overview
This week focuses on setting up your practice environment and understanding Kubernetes fundamentals.

## Daily Tasks

### Day 1: Linux Setup & CKA Environment (1.5 hours)
- [x] Install Linux on 3 VMs
- [x] Configure SSH and essential tools
- [x] Set up bash aliases: `alias k=kubectl`
- [x] Enable kubectl bash completion

### Day 2: Container Runtime Foundation (1.5 hours)
- [ ] Install Podman and CRI-O
- [ ] Practice basic Podman commands: `run`, `ps`, `logs`, `exec`
- [ ] Understand container runtime in Kubernetes context

### Day 3: Kubernetes Architecture Deep Dive (1.5 hours)
- [ ] Study control plane components (API server, etcd, scheduler)
- [ ] Study worker node components (kubelet, kube-proxy)
- [ ] Document architecture for troubleshooting reference

### Day 4: kubectl Installation & Basics (1.5 hours)
- [ ] Install kubeadm, kubelet, kubectl on all nodes
- [ ] Practice basic kubectl commands and output formats
- [ ] Master `kubectl explain` for API reference

### Day 5: First Cluster Setup (1.5 hours)
- [ ] Complete `kubeadm init` process
- [ ] Understand cluster certificates and tokens
- [ ] Document setup steps for speed improvement

## Weekend Session (3 hours)
- [ ] Complete cluster setup from scratch (multiple attempts)
- [ ] Join worker nodes and practice basic troubleshooting
- [ ] **Goal Check:** Cluster setup in under 30 minutes

## Key Commands to Practice

```bash
# Cluster initialization
kubeadm init --pod-network-cidr=10.244.0.0/16

# Join worker nodes
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# Basic kubectl commands
kubectl get nodes
kubectl get pods -A
kubectl describe node <node-name>
kubectl cluster-info

# Container runtime commands
podman run -d nginx
podman ps
podman logs <container-id>
podman exec -it <container-id> bash
```

## Learning Resources
- Kubernetes Architecture: https://kubernetes.io/docs/concepts/architecture/
- kubeadm documentation: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/
- Container Runtime Interface: https://kubernetes.io/docs/concepts/architecture/cri/

## Success Criteria
- [ ] Can set up a multi-node cluster from scratch
- [ ] Understand all Kubernetes components and their roles
- [ ] Comfortable with basic kubectl operations
- [ ] Can troubleshoot basic cluster issues

## Notes Section
_Use this space to document important learnings, commands, and troubleshooting tips_