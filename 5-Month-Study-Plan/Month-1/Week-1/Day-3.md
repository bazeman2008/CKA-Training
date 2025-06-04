# Day 3: Kubernetes Architecture Deep Dive

## Overview (1.5 hours)
Master Kubernetes architecture components to build troubleshooting skills and understand how the system works end-to-end.

## Why This Matters for the CKA Exam
- **25% of exam** covers Cluster Architecture, Installation & Configuration
- **30% troubleshooting questions** require understanding component interactions
- **Component failure scenarios** are common exam questions
- **Architecture knowledge** enables systematic problem-solving

## Task 1: Study Control Plane Components (45 minutes)

### What You'll Do
Deep dive into each control plane component and understand their roles in cluster operations.

### API Server (kube-apiserver)

#### Purpose and Function
```bash
# The API server is the front-end to the Kubernetes control plane
# ALL communication goes through the API server

# Key responsibilities:
# 1. Validates and processes REST operations
# 2. Updates etcd with cluster state
# 3. Provides authentication and authorization
# 4. Serves the Kubernetes API
```

#### CKA Exam Scenarios
- **API Server Down**: Cluster becomes unresponsive
- **Certificate Issues**: Authentication failures
- **Port Conflicts**: Service binding problems
- **Configuration Errors**: Cluster initialization failures

#### Troubleshooting Commands
```bash
# Check API server status
kubectl cluster-info
curl -k https://localhost:6443/healthz

# View API server logs (when cluster is up)
kubectl logs -n kube-system kube-apiserver-<master-node>

# Check API server process (direct)
ps aux | grep kube-apiserver
sudo systemctl status kube-apiserver  # If using systemd

# API server configuration location
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

### etcd (Cluster Data Store)

#### Purpose and Function
```bash
# etcd stores all cluster data
# Single source of truth for cluster state

# Key responsibilities:
# 1. Store cluster configuration
# 2. Store cluster state
# 3. Watch for changes
# 4. Provide consistent data access
```

#### CKA Exam Critical Skills
```bash
# etcd backup (CRITICAL for exam)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# etcd restore (CRITICAL for exam)
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore

# Check etcd health
ETCDCTL_API=3 etcdctl endpoint health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

### Scheduler (kube-scheduler)

#### Purpose and Function
```bash
# Scheduler assigns pods to nodes
# Makes placement decisions based on resources and constraints

# Key responsibilities:
# 1. Find suitable nodes for pods
# 2. Consider resource requirements
# 3. Apply scheduling policies
# 4. Handle affinity/anti-affinity rules
```

#### CKA Exam Scenarios
- **Pods Pending**: Scheduler can't find suitable nodes
- **Resource Constraints**: Insufficient CPU/memory
- **Taints and Tolerations**: Scheduling restrictions
- **Node Affinity**: Placement requirements

#### Troubleshooting Commands
```bash
# Check scheduler status
kubectl get pods -n kube-system | grep scheduler
kubectl logs -n kube-system kube-scheduler-<master-node>

# Check pending pods (scheduler issues)
kubectl get pods --all-namespaces | grep Pending
kubectl describe pod <pending-pod> | grep Events -A 10

# Check node resources
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"
```

### Controller Manager (kube-controller-manager)

#### Purpose and Function
```bash
# Runs controllers that handle routine cluster tasks
# Multiple controllers in one binary

# Key controllers:
# 1. Node Controller - monitors node health
# 2. Replication Controller - maintains pod replicas
# 3. Service Controller - manages service endpoints
# 4. Deployment Controller - handles deployments
```

#### CKA Exam Scenarios
- **Pods Not Recreated**: Replication controller issues
- **Services Not Working**: Service controller problems
- **Node Issues**: Node controller not responding
- **Deployment Failures**: Deployment controller errors

#### Troubleshooting Commands
```bash
# Check controller manager
kubectl logs -n kube-system kube-controller-manager-<master-node>

# Check controller status
kubectl get pods -n kube-system | grep controller-manager

# Check specific controller issues
kubectl get events --all-namespaces | grep controller
```

## Task 2: Study Worker Node Components (45 minutes)

### What You'll Do
Understand worker node components and their interaction with control plane.

### kubelet (Node Agent)

#### Purpose and Function
```bash
# kubelet is the primary node agent
# Communicates with API server and manages containers

# Key responsibilities:
# 1. Register node with cluster
# 2. Monitor pod specifications from API server
# 3. Ensure containers are running in pods
# 4. Report node and pod status back
# 5. Run liveness and readiness probes
```

#### CKA Exam Critical Skills
```bash
# Check kubelet status
sudo systemctl status kubelet
journalctl -u kubelet -f

# View kubelet configuration
sudo cat /var/lib/kubelet/config.yaml
ps aux | grep kubelet

# Common kubelet issues
# 1. Certificate problems
# 2. Network configuration
# 3. Container runtime connectivity
# 4. Resource constraints
```

#### Troubleshooting Node Issues
```bash
# Node troubleshooting workflow
kubectl get nodes                    # Check node status
kubectl describe node <node-name>    # Detailed node info
kubectl get events | grep <node-name> # Node-related events

# Common node problems:
# - NotReady status
# - MemoryPressure
# - DiskPressure  
# - PIDPressure
```

### kube-proxy (Network Proxy)

#### Purpose and Function
```bash
# kube-proxy handles service networking
# Implements services and load balancing

# Key responsibilities:
# 1. Maintain network rules for services
# 2. Perform load balancing across pod endpoints
# 3. Handle service discovery
# 4. Manage iptables/ipvs rules
```

#### CKA Exam Scenarios
- **Service Connectivity Issues**: kube-proxy problems
- **Load Balancing Failures**: Endpoint management
- **Network Policy Problems**: Traffic routing issues

#### Troubleshooting Commands
```bash
# Check kube-proxy status
kubectl get pods -n kube-system | grep kube-proxy
kubectl logs -n kube-system kube-proxy-<node-name>

# Check service endpoints
kubectl get endpoints
kubectl describe service <service-name>

# Network troubleshooting
kubectl get services --all-namespaces
netstat -tulpn | grep kube-proxy
```

### Container Runtime (CRI)

#### Purpose and Function
```bash
# Container runtime runs containers
# Interfaces with kubelet through CRI

# Key responsibilities:
# 1. Pull container images
# 2. Run containers
# 3. Monitor container health
# 4. Report container status
```

#### CKA Exam Integration
```bash
# Runtime commands for troubleshooting
sudo crictl ps                    # List containers
sudo crictl pods                  # List pods
sudo crictl images               # List images
sudo crictl logs <container-id>  # Container logs
```

## Task 3: Document Architecture for Troubleshooting Reference (20 minutes)

### What You'll Do
Create a troubleshooting reference document for quick consultation during the exam.

### Component Interaction Map
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   kubectl       │────│  API Server     │────│     etcd        │
│   (client)      │    │ (kube-apiserver)│    │ (data store)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              │
                    ┌─────────┴─────────┐
                    │                   │
            ┌───────▼────────┐ ┌────────▼────────┐
            │   Scheduler    │ │ Controller Mgr  │
            │(kube-scheduler)│ │(kube-controller)│
            └────────────────┘ └─────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │                   │
            ┌───────▼────────┐ ┌────────▼────────┐
            │    kubelet     │ │   kube-proxy    │
            │  (node agent)  │ │(network proxy)  │
            └────────────────┘ └─────────────────┘
                    │
            ┌───────▼────────┐
            │Container Runtime│
            │   (CRI-O)      │
            └────────────────┘
```

### Troubleshooting Decision Tree
```bash
# Problem: Cluster not responding
1. Check API Server: kubectl cluster-info
2. Check etcd: etcdctl endpoint health
3. Check certificates: openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text

# Problem: Pods not scheduling
1. Check scheduler: kubectl logs -n kube-system kube-scheduler-*
2. Check node resources: kubectl describe nodes
3. Check pod requirements: kubectl describe pod <pod-name>

# Problem: Pods not starting
1. Check kubelet: systemctl status kubelet
2. Check container runtime: systemctl status crio
3. Check image pull: kubectl describe pod <pod-name>

# Problem: Services not working
1. Check kube-proxy: kubectl logs -n kube-system kube-proxy-*
2. Check endpoints: kubectl get endpoints
3. Check network policies: kubectl get networkpolicy
```

### Quick Reference Commands
```bash
# Control Plane Health Check
kubectl get componentstatuses
kubectl cluster-info
kubectl get pods -n kube-system

# Node Health Check  
kubectl get nodes
kubectl top nodes
kubectl describe node <node-name>

# Service Health Check
kubectl get services --all-namespaces
kubectl get endpoints
netstat -tulpn | grep :6443
```

## CKA Exam Application

### High-Frequency Exam Scenarios

#### Scenario 1: API Server Certificate Issues
```bash
# Problem: kubectl commands failing with certificate errors
# Solution approach:
1. Check certificate expiration: kubeadm certs check-expiration
2. Renew certificates: kubeadm certs renew all
3. Restart API server: systemctl restart kubelet
```

#### Scenario 2: etcd Backup and Restore
```bash
# This is almost guaranteed to be on the exam
# Practice until you can do it in under 10 minutes

# Backup
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Restore
1. Stop API server: mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp
2. Restore data: ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd.db --data-dir=/var/lib/etcd-new
3. Update etcd manifest: vim /etc/kubernetes/manifests/etcd.yaml (change data-dir)
4. Start API server: mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

#### Scenario 3: Node Troubleshooting
```bash
# Problem: Node showing NotReady status
# Solution approach:
1. Check node: kubectl describe node <node-name>
2. Check kubelet: systemctl status kubelet
3. Check logs: journalctl -u kubelet -f
4. Check resources: df -h && free -h
```

## Time Management for CKA

### Architecture Knowledge Speed Tips
- Memorize component locations and log paths
- Practice troubleshooting workflows repeatedly
- Know the difference between systemd and static pod components
- Understand when to use kubectl vs direct system commands

### Common Time Wasters
- Looking up basic component functions
- Not knowing log locations
- Forgetting certificate paths for etcd commands
- Not understanding component dependencies

## Daily Goals Checklist

- [ ] Understand all control plane components and their functions
- [ ] Know all worker node components and their roles
- [ ] Can troubleshoot each component independently
- [ ] Have created architecture reference document
- [ ] Practiced etcd backup/restore commands
- [ ] Can identify component issues quickly

## Next Day Preparation

### What You'll Need for Day 4
- Solid architecture understanding ✓
- Component troubleshooting skills ✓
- Reference documentation created ✓

### Coming Up: kubectl Installation & Basics
Day 4 will install kubectl and practice the commands you'll use throughout the exam.

## Study Notes Section

**Component Quick Reference:**
```bash
# Control Plane Logs
kubectl logs -n kube-system kube-apiserver-*
kubectl logs -n kube-system kube-scheduler-*
kubectl logs -n kube-system kube-controller-manager-*
kubectl logs -n kube-system etcd-*

# Worker Node Logs
journalctl -u kubelet
kubectl logs -n kube-system kube-proxy-*
systemctl status crio
```

**Personal Architecture Notes:**
_Document any specific insights about component interactions or troubleshooting approaches_